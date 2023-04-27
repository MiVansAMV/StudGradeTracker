package main

import (
	"fmt"
	"os"

	"github.com/go-redis/redis"
)

type Student struct {
	Name   string
	Course string
	Grades []int
}

func main() {
	// Connect to Redis
	client := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "", // no password set
		DB:       0,  // use default DB
	})

	// Test Redis connection
	pong, err := client.Ping().Result()
	if err != nil {
		fmt.Println("Error connecting to Redis:", err)
		os.Exit(1)
	}
	fmt.Println("Connected to Redis:", pong)

	// Create a new student
	student := Student{
		Name:   "John Doe",
		Course: "Math",
		Grades: []int{80, 90, 85},
	}

	// Save student grades to Redis
	err = client.HMSet(student.Name, map[string]interface{}{
		"course": student.Course,
		"grades": student.Grades,
	}).Err()
	if err != nil {
		fmt.Println("Error saving grades to Redis:", err)
	}

	// Retrieve student grades from Redis
	result, err := client.HGetAll(student.Name).Result()
	if err != nil {
		fmt.Println("Error retrieving grades from Redis:", err)
	}

	// Print student grades
	fmt.Printf("Student: %s\n", student.Name)
	fmt.Printf("Course: %s\n", result["course"])
	fmt.Printf("Grades: %v\n", result["grades"])
}
