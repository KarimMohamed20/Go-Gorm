# installation
 * In order to install the jinzhu/gorm, you will have to perform the following go get command:

``
go get -u github.com/jinzhu/gorm
``
 * After you’ve done this, you should be able to import the jinzhu/gorm into any of your go based projects.

# A Simple Example
### Say, for instance, you wanted to write a go REST API that saved new users and their emails to a database when a certain API endpoint was hit.

 ## We could describe our users in a go struct like so:

// Our User Struct
``` go
type User struct {
	gorm.Model
	Name  string
	Email string
}
```
 * Once we have defined our User model we can then go about exposing an API endpoint that could save new users to our sqlite3 database.

 * If you want to know how you can develop your own Go based REST API, check out my other Go Tutorial: Building a RESTful API in Go

 ### Our API
 * So, we are going to create a very simple API which features 4 distinct CRUD endpoints. These will return all users, add a new user, delete a user and update a user.

 * With the help of our new GORM, the creation of these endpoints should be far simpler than they would have been, should we have went down a standard raw SQL route.

``` go 
package main

import (
	"fmt"
	"log"
	"net/http"

	"github.com/gorilla/mux"
)

func allUsers(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "All Users Endpoint Hit")
}

func newUser(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "New User Endpoint Hit")
}

func deleteUser(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Delete User Endpoint Hit")
}

func updateUser(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Update User Endpoint Hit")
}

func handleRequests() {
	myRouter := mux.NewRouter().StrictSlash(true)
	myRouter.HandleFunc("/users", allUsers).Methods("GET")
	myRouter.HandleFunc("/user/{name}", deleteUser).Methods("DELETE")
	myRouter.HandleFunc("/user/{name}/{email}", updateUser).Methods("PUT")
	myRouter.HandleFunc("/user/{name}/{email}", newUser).Methods("POST")
	log.Fatal(http.ListenAndServe(":8081", myRouter))
}


func main() {
	fmt.Println("Go ORM Tutorial")

	// Handle Subsequent requests
	handleRequests()
}
```
 * We can then start this new API by running go run main.go. This API represents the base from which we will build our ORM based solution.

 * SQLite3 Database Creation and Automatic Schema Migration
 * The next step of our project is creating a database. For the purpose of this tutorial, we are going to use a sqlite3 database due to its ease of use and setup.

 * Note: You can swap to using another database technology fairly easily using the GORM by switching dialects.

 * We can use GORM to automatically create the User table within our database by calling db.AutoMigrate(&User{}). This saves us the hassle of writing a table creation SQL script.

// our initial migration function
``` go
func initialMigration() {
	db, err := gorm.Open("sqlite3", "test.db")
	if err != nil {
		fmt.Println(err.Error())
		panic("failed to connect database")
	}
	defer db.Close()

	// Migrate the schema
	db.AutoMigrate(&User{})
}

func main() {
	fmt.Println("Go ORM Tutorial")
    
    // Add the call to our new initialMigration function
	initialMigration()
	
    handleRequests()
}
```
 * Updating our All Users Endpoint
 * Within our allUsers() function we basically want to query for all the User records within our database and then encode this into JSON and return this as the response.

 * We can query all of the users within our database by calling db.Find(&users).

``` go
func allUsers(w http.ResponseWriter, r *http.Request) {
	db, err := gorm.Open("sqlite3", "test.db")
	if err != nil {
		panic("failed to connect database")
	}
	defer db.Close()

	var users []User
	db.Find(&users)
	fmt.Println("{}", users)

	json.NewEncoder(w).Encode(users)
}
```
 * Updating our New User Endpoint
 * We now want to update our newUser() function so that it can insert new users into our database. This will need to parse both a username and an email from the query parameters of the request made to our API.

 * This will have to parse the path params of our endpoint and then use these path params to populate a new User object that we will then insert into our sqlite database by calling db.Create(&User{Name: name, Email: email}) like so:

``` go
func newUser(w http.ResponseWriter, r *http.Request) {
	fmt.Println("New User Endpoint Hit")

	db, err := gorm.Open("sqlite3", "test.db")
	if err != nil {
		panic("failed to connect database")
	}
	defer db.Close()

	vars := mux.Vars(r)
	name := vars["name"]
	email := vars["email"]

	db.Create(&User{Name: name, Email: email})
	fmt.Fprintf(w, "New User Successfully Created")
}
```
 * Our Delete User Endpoint
 * Our deleteUser() function will delete a user that matches the same name passed into it via a path parameter. It’s rather basic and doesn’t handle the cases where more than one user exists within the database with the same but it serves a good example in this project.

``` go
func deleteUser(w http.ResponseWriter, r *http.Request) {
	db, err := gorm.Open("sqlite3", "test.db")
	if err != nil {
		panic("failed to connect database")
	}
	defer db.Close()

	vars := mux.Vars(r)
	name := vars["name"]

	var user User
	db.Where("name = ?", name).Find(&user)
	db.Delete(&user)

	fmt.Fprintf(w, "Successfully Deleted User")
}
```
Our Update User Endpoint
On the odd occasion that you need to update an existing user within your database, you can certainly do that in a far easier fashion using the GORM. Essentially what you have to do is to search for a given user using a unique name.

Once you have this user, you simply update the User object as you normally would a standard go object. Once you are happy with the object and your updates you then call db.Save(&user) to save any changes to the database.

``` go
func updateUser(w http.ResponseWriter, r *http.Request) {
	db, err := gorm.Open("sqlite3", "test.db")
	if err != nil {
		panic("failed to connect database")
	}
	defer db.Close()

	vars := mux.Vars(r)
	name := vars["name"]
	email := vars["email"]

	var user User
	db.Where("name = ?", name).Find(&user)

	user.Email = email

	db.Save(&user)
	fmt.Fprintf(w, "Successfully Updated User")
}
```

