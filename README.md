Documentation for the "Current Toronto Time API" Application
Overview
This application is a simple API built using Go, which returns the current time in Toronto adjusted for its local timezone. Each time a request is made to the /current-time endpoint, the API logs the current time into a MySQL database. This allows us to keep track of all time requests made through the API.
Deliverables
1.	Source Code of the Go Application
o	The main source file for this application is main.go. This file contains all the necessary logic to start a web server, handle API requests, connect to the MySQL database, and log the time of each request.
2.	Instructions for Setting Up and Running the Application
o	These steps will guide you through the process of setting up and running the application on your local machine.
3.	Script/Instructions for Setting Up the MySQL Database and Table
o	Here, you will find instructions for setting up MySQL using Docker and creating the necessary database and table.

Instructions for Setting Up and Running the Go Application
Step 1: Install Dependencies
Make sure that Go is installed on your system. You can download and install Go from the official website.
Step 2: Initialize a Go Project
1.	Open your terminal (Git Bash, VS Code terminal, etc.) and navigate to your project directory.
2.	Run the following command to initialize a new Go module:
(GitBash) go mod init toronto-time-api
Step 3: Install MySQL Driver for Go
Run the following command to install the required MySQL driver:
(GitBash) go get -u github.com/go-sql-driver/mysql
Step 4: Add the Code
Create a new file named main.go in your project directory. Add the following code to handle the API and database operations:
Go
package main

import (
    "database/sql"
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "os"
    "time"

    _ "github.com/go-sql-driver/mysql"
)

var db *sql.DB

func main() {
    // Connect to MySQL
    var err error
    dsn := "root:password@tcp(localhost:3306)/toronto_time"
    db, err = sql.Open("mysql", dsn)
    if err != nil {
        log.Fatalf("Error connecting to database: %v", err)
    }
    defer db.Close()

    // Test the connection
    if err = db.Ping(); err != nil {
        log.Fatalf("Error pinging database: %v", err)
    }

    // Set up HTTP server
    http.HandleFunc("/current-time", handleCurrentTime)
    fmt.Println("Server is running on http://localhost:8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}

func handleCurrentTime(w http.ResponseWriter, r *http.Request) {
    // Get Toronto time
    loc, err := time.LoadLocation("America/Toronto")
    if err != nil {
        http.Error(w, "Failed to load time zone", http.StatusInternalServerError)
        return
    }
    currentTime := time.Now().In(loc)

    // Log time to database
    _, err = db.Exec("INSERT INTO time_log (timestamp) VALUES (?)", currentTime)
    if err != nil {
        http.Error(w, "Failed to log time to database", http.StatusInternalServerError)
        return
    }

    // Respond with JSON
    response := map[string]string{"current_time": currentTime.Format(time.RFC3339)}
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}
Step 5: Run the Application
1.	After saving your main.go file, run the following command in your terminal:
(GitBash) go run main.go
2.	The server will be available at http://localhost:8080. You can test it using your browser, Postman, or curl:
(GitBash) curl http://localhost:8080/current-time

Script/Instructions for Setting Up the MySQL Database and Table
Step 1: Install Docker
If you don't have MySQL installed, you can use Docker to run MySQL in a container. Install Docker Desktop.
Step 2: Start MySQL in Docker
Run the following command to start a MySQL container in Docker:
(GitBash) docker run --name mysql-container -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=toronto_time -p 3306:3306 -d mysql:8.0
Explanation:
•	--name mysql-container: Names the container.
•	-e MYSQL_ROOT_PASSWORD=password: Sets the root password for MySQL.
•	-e MYSQL_DATABASE=toronto_time: Creates a database named toronto_time.
•	-p 3306:3306: Maps MySQL's default port (3306) to your local machine.
•	-d mysql:8.0: Runs MySQL in the background using the MySQL 8.0 image.
Step 3: Verify the MySQL Container is Running
Check if the MySQL container is running by using:
(GitBash) docker ps
You should see mysql-container listed.
Step 4: Access the MySQL Shell
To access the MySQL shell inside the container, run:
(GitBash) docker exec -it mysql-container mysql -h 127.0.0.1 -u root -p
Enter the root password (password) when prompted.

Step 5: Create the time_log Table
Once you're inside the MySQL shell, run the following SQL commands to create the time_log table:
sql
USE toronto_time;
CREATE TABLE time_log (
    id INT AUTO_INCREMENT PRIMARY KEY,
    timestamp DATETIME NOT NULL
);
Exit the MySQL shell by typing:
EXIT;

Conclusion
With the Go API and MySQL database set up, your application is now ready to return the current time in Toronto and log every request to the database. By following these instructions, you'll be able to run the application locally and ensure everything works as expected.
