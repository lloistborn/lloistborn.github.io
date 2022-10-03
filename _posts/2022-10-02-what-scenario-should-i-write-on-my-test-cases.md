## intent

## problem

## example
Sample of business layer that we want to test are having dependencies to a repo and networking layer.

*application structure*
```bash
├── Application
|   ├── Services                    
|   │   ├── CoordinateService.cs
|   ├── Repository                  
|   │   ├── CoordinateRepo.cs
|   ├── HttpClient
|   │   ├── CoordinateClient.cs          
```
*process*

The image below shows how `CoordinateService.cs` is communicating with `Repository` and `HttpClient`

![img](/assets/images/app-structure.png)

*code*

```c#
public class ICoordinateHandler {
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


Firstly,lets setup the test class

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
public Task TestGetCoordinateDetails_ReturnLocationDetails() {

}
```

Based on the logic here are the negative cases:
1. TestGetCoordinateDetails_WhenLocationHasNotFoundFromHttpClient_ReturnLocationNotFound
2. TestGetCoordinateDetails_WhenLocationHasNotFoundInDb_ReturnLocationNotFound

Any other negative cases like timeout issue or DB connection issue will be covered on the Integration Tests post.

```c#
@Test
public Task TestGetCoordinateDetails_WhenLocationHasNotFoundFromHttpClient_ReturnLocationNotFound() {
    
}


@Test
public Task TestGetCoordinateDetails_WhenLocationHasNotFoundInDb_ReturnLocationNotFound() {
    
}
```

## rules of thumbs