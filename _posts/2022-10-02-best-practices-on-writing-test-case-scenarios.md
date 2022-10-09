## intent
This post will go through most of scenarios of how to satisfy the requirement of unit tests and example of test cases that are considered as important.

## problem
Using this application structure, we should focus on writing unit tests for the `Application`'s layer only, since this layer is where we build our business logic. Other layer should not be included when writing unit tests. 

In some events, you might think unit test is also important for other layer like `Infrastructure`. In this case, that means you have a business logic spread on other layer than `Application`, which is not ideal. 

```bash
├── Insfrastructure
│   ├── Repository
│   ├── Controller
├── Middleware
│   ├── Exception Logic
│   ├── Logger
├── Application
│   ├── Business Layer
│   ├── Utilities
```


## example
Sample of business layer that we want to test are having dependencies to a repo and networking layer.

*application structure*
```bash
├── Application
│   ├── Services                    
│   │   ├── CoordinateService.cs
├── Repository                  
│   ├── CoordinateRepo.cs
├── HttpClient
│   ├── CoordinateClient.cs          
```
*process*

The image below shows how `CoordinateService.cs` is communicating with `Repository` and `HttpClient`

![img](/assets/images/app-structure.png)

*code*

```c#
public interface ICoordinateHandler {
    Task<Coordinate> GetCoordinateDetails(double lng, double lat);
}

public class CoordinateService : ICoordinateHandler {
    private ICoordinateRepo _repo;
    private ICoordinateClient _client;

    public CoordinateService(ICoordinateRepo repo, ICoordinateClient client) {
        _repo = repo;
        _client = client;
    }

    public async Task<Coordinate> GetCoordinateDetails(double lng, double lat) {
        Coordinate location = await _client.getLocation(lng, lat);
        
        if (location.hasFound()) {
            return await _repo.getLocationDetails(location.area_id);
        }
        
        return new location.hasNotFound();
    }
}
```

*unit tests*

Let's make the unit test straightforward by focusing only on the important part which is to test the main logic of `Coordinate GetCoordinateDetails(double lng, double lat)`

The function `GetCoordinateDetails` does exactly this purpose:

> Given coordinate longitude and latitude with any decimal numbers, search for its location details


Firstly, lets setup the test class

```c#
public class CoordinateServiceTests {
    private Mock<ICoordinateRepo> _mockRepo;
    private Mock<ICoordinateClient> _mockClient;

    private CoordinateService service;

    @Setup
    public void setup() {
        _mockRepo = new Mock<ICoordinateRepo>();
        _mockClient = new Mock<ICoordinateClient>();

        service = new CoordinateService(_mockRepo.Object, _mockClient.Object);
    }
}
```
Notice that, we are mocking the `ICoordinateRepo` and the `ICoordinateClient`. This is because we don't want to deal with its logic and just want to reproduce multiple scenarios that might occurs based on the mocked result.


On that sense, here is the normal happy case:
```c#
@Test
public Task TestGetCoordinateDetails_ThenReturnLocationDetails() {
    // arrange
    var coordinate = new Coordinate() { 
        hasFound: true,
        area_id:  1123
    };

    _mockClient.Setup(f => f.getLocation(It.IsAny<double>(), It.IsAny<double>()))
        .ReturnsAsync(coordinate);
    _mockRepo.Setup(f => f.getLocationDetails(coordinate.area_id))
        .ReturnAsync(new Coordinate() {hasFound: true, area_id: 1123, area_name: "downtown"});

    // act
    var actualResult = await service.GetCoordinateDetails(24.33, 55.00);

    // asert
    Assert.IsFalse(actualResult.hasNotFound());
}
```

The function format naming is also important part of writing test function. This is because unit test also played role as self documentation. It is explaining what to expect from the function being test.

The function is usually using this format that make it easy to understand:
```
1. TestFunctionName
2. ThenWhatWillHappen

void TestFunctionName_ThenWhatWillHappen()
```

Based on the logic in the normal flow, here are the negative cases:
1. TestGetCoordinateDetails_WhenLocationHasNotFoundFromHttpClient_ReturnLocationNotFound
2. TestGetCoordinateDetails_WhenLocationHasNotFoundInDb_ReturnLocationNotFound

Any other negative cases like timeout or DB connection issue should not be included here as we are only focusing on the business logic.

```c#
@Test
public Task TestGetCoordinateDetails_WhenLocationHasNotFoundFromHttpClient_ReturnLocationNotFound() {
    // arrange
    var coordinate = new Coordinate() { 
        hasFound: false
    };

    _mockClient.Setup(f => f.getLocation(It.IsAny<double>(), It.IsAny<double>()))
        .ReturnsAsync(coordinate);

    // act
    var actualResult = await service.GetCoordinateDetails(24.33, 55.00);

    // asert
    Assert.IsTrue(actualResult.hasNotFound());    
}


@Test
public Task TestGetCoordinateDetails_WhenLocationHasNotFoundInDb_ReturnLocationNotFound() {
    // arrange
    var coordinate = new Coordinate() { 
        hasFound: true,
        area_id:  1123
    };

    _mockClient.Setup(f => f.getLocation(It.IsAny<double>(), It.IsAny<double>()))
        .ReturnsAsync(coordinate);
    _mockRepo.Setup(f => f.getLocationDetails(coordinate.area_id))
        .ReturnAsync(new Coordinate() {hasNotFound: true);

    // act
    var actualResult = await service.GetCoordinateDetails(24.33, 55.00);

    // asert
    Assert.IsTrue(actualResult.hasNotFound());
    
}
```

## rules of thumbs
*Do's*

✅ Always write unit tests for the business logic.

✅ Start with happy-normal case then continue with negative cases.

✅ Create clear function name for each unit test by following actual flow of the code.

✅ Use the AAA format for the test function.

✅ Write as many as possible test cases.

*Dont's*

🙅‍♂️ Write unit tests for other layer than the Business.

🙅‍♂️ Not providing enough negative cases.

🙅‍♂️ Calling too many function being tests.