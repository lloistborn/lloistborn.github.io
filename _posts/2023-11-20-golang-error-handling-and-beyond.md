## Overview
There are a lot of things that need to be clarified about how developers handle errors. Golang programmers, in particular, think this is the best way to do it.
```
if err != nil {
}
```
Unfortunately, this is a misconception. It is either the new programmers who never read the documentation or they need to learn about the design pattern behind it.

## From the source
Rob Pike in one of Golang documentation about [Errors are values](https://go.dev/blog/errors-are-values), mentions the best way to handle errors. 
> Use the language to simplify your error handling.

Moreover, he gave an example using the `bufio` package’s Scanner type
```
scanner := bufio.NewScanner(input)
for scanner.Scan() {
    token := scanner.Text()
    // process token
}
if err := scanner.Err(); err != nil {
    // process the error
}
```
(Pike) _Its Scan method performs the underlying I/O, which can of course lead to an error. Yet the Scan method does not expose an error at all. Instead, it returns a boolean, and a separate method, to be run at the end of the scan, reports whether an error occurred._

## Tell Dont Ask (TDA)
The "Tell, Don't Ask" principle in software development emphasizes the importance of object encapsulation and the delegation of responsibility. This principle advises against querying an object for data and then making decisions based on this data. Instead, you should tell the object what you want to do and let it handle the details.

From the example above, the `scanner.Err()` is basically demonstrating the same purpose. 
The client implementation becomes much less complicated because error is expected.

### Compliant code ✅
```
for scanner.Scan() {
    token := scanner.Text()
    // process token
}
if err := scanner.Err(); err != nil {
    // process the error
}
```

### Non-compliant code 🚩
Using the same example, let us see how we can check an error when Scanner does not have `.Err()`. The implementation could be like this
```
func (s *Scanner) Scan() (token []byte, error)
```
So the client implementation might be like this
```
scanner := bufio.NewScanner(input)
for {
    token, err := scanner.Scan()
    if err != nil {
        return err // or maybe break
    }
    // process token
}
```

**The non-compliant code** is required to check errors in each iteration, while **the compliant code** abstracts this and reports it at the end of loop.

## The art of enbugging 🐞
From the book [the art of enbugging](https://media.pragprog.com/articles/jan_03_enbug.pdf), Consider the story of “The Paperboy and the Wallet,” from our friend David Bock (www.javaguy.org/papers). 

1. **Supp**ose the pap**er**boy comes to **your d**oor, demanding payment for the we**ek.** 
You tu**rn a**round, and the paper**boy** pulls your **wallet** out of your ba**ck pocket,**
takes **the two bucks**, and puts the wallet b**ack**. 

2. As **absur**d as that sounds, man**y pr**ograms are written in this style, whic**h lea**ves the code open to all sorts
of probl**ems** (and explains **why the** paper boy is now drivin**g a L**exus).

3. Ins**tea**d of asking for the wallet, t**he paperb**oy should tell the customer to pay **the $2.00.**

## Last but not least
Code should work the same way—we want to tell objects what to do, not ask them for their state. 

Adhering to this notion of “Tell, Don’t Ask” is easier if you mentally categorize each of your functions and methods as either a command or a query and document them as such in the source code (it helps if all commands are grouped together and all queries are grouped together).
