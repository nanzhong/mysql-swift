mysql-swift
===========

[![Swift 4.1](https://img.shields.io/badge/Swift-4.1-orange.svg)](https://swift.org)
![Platform Linux, macOS](https://img.shields.io/badge/Platforms-Linux%2C%20macOS-lightgray.svg)
[![CircleCI](https://circleci.com/gh/novi/mysql-swift.svg?style=svg)](https://circleci.com/gh/novi/mysql-swift)



MySQL client library for Swift.
This is inspired by Node.js' [mysql](https://github.com/felixge/node-mysql).

* Based on libmysqlclient
* Raw SQL query
* Simple query formatting and escaping (same as Node's)
* Mapping queried results to `Codable` structs or classes

_Note:_ No asynchronous support currently. It depends libmysqlclient.

```swift
// Declare a model

struct User: Codable, QueryParameter {
    let id: Int
    let userName: String
    let age: Int?
    let status: Status
    let createdAt: Date
    
    enum Status: String, Codable {
        case created = "created"
        case verified = "verified"
    }
    
    private enum CodingKeys: String, CodingKey {
        case id
        case userName = "user_name"
        case age
        case status = "status"
        case createdAt = "created_at"
    }
}
    
// Selecting
let nameParam = "some one"
let ids: [QueryParameter] = [1, 2, 3, 4, 5, 6]
let optionalInt: Int? = nil
let rows: [User] = try conn.query("SELECT id,user_name,status,status,created_at FROM `user` WHERE (age > ? OR age is ?) OR name = ? OR id IN (?)", [50, optionalInt, nameParam, QueryArray(ids)] ])

// Inserting
let age: Int? = 26
let user = User(id: 0, userName: "novi", age: age, status: .created, createdAt: Date())
let status = try conn.query("INSERT INTO `user` SET ?", [user]) as QueryStatus
let newId = status.insertedId

// Updating
let defaultAge = 30
try conn.query("UPDATE `user` SET age = ? WHERE age is NULL;", [defaultAge])

``` 

# Requirements

* Swift 4.1 or later

# Dependencies

* MariaDB or MySQL Connector/C (libmysqlclient) 2.2.3

## macOS

Install pkg-config `.pc` in [cmysql](https://github.com/vapor-community/cmysql) or [cmysql-mariadb](https://github.com/novi/cmysql-mariadb/tree/mariadb).

```
brew install https://gist.github.com/novi/dd21d48d260379e8919d9490bf5cfaec/raw/6ea4daa02d93f4ab0110ad30d87ea2b497a71cd0/cmysqlmariadb.rb
```

## Ubuntu Linux

* Install `libmariadbclient`
* Follow [Setting up MariaDB Repositories](https://downloads.mariadb.org/mariadb/repositories/#mirror=yamagata-university) and set up your repository.

```sh
$ sudo apt-get install libmariadbclient-dev
```

# Installation

## Swift Package Manager

* Add `mysql-swift` to `Package.swift` of your project.

```swift
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    ...,
    dependencies: [
        .package(url: "https://github.com/novi/mysql-swift.git", .upToNextMinor(from: "0.9.0"))
    ],
    targets: [
        .target(
            name: "YourAppOrLibrary",
            dependencies: [
                // add a dependency
                "MySQL", 
            ]
        )
    ]
)
```

# Usage

## Connection & Querying

1. Create a pool with options (hostname, port, password,...).
2. Use `pool.execute()`. It automatically get and release a connection. 

```swift
let options = Options(host: "your.mysql.host"...)
let pool = ConnectionPool(options: options) // Create pool with options
let rows: [User] = try pool.execute { conn in
	// The connection is held in this block
	try conn.query("SELECT * FROM users;") // And it returns result to outside execute block
}
```

## Transaction

```swift	
let wholeStaus: QueryStatus = try pool.transaction { conn in
	let status = try conn.query("INSERT INTO users SET ?;", [user]) as QueryStatus // Create a user
	let userId = status.insertedId // the user's id
	try conn.query("UPDATE info SET val = ? WHERE key = 'latest_user_id' ", [userId]) // Store user's id that we have created the above
}
wholeStaus.affectedRows == 1 // true
```



# License

MIT
