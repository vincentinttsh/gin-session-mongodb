# gin-session-mongodb
[![Build Status](https://travis-ci.com/vincentinttsh/gin-session-mongodb.svg?branch=master)](https://travis-ci.com/vincentinttsh/gin-session-mongodb)
[![Go Report Card](https://goreportcard.com/badge/github.com/vincentinttsh/gin-session-mongodb)](https://goreportcard.com/report/github.com/vincentinttsh/gin-session-mongodb)
[![codecov](https://codecov.io/gh/vincentinttsh/gin-session-mongodb/branch/master/graph/badge.svg?token=IWORJUODXQ)](https://codecov.io/gh/vincentinttsh/gin-session-mongodb)

Gin middleware for session management with MongoDB (MongoDB Go Driver) support

## Installation

```
go get github.com/vincentinttsh/gin-session-mongodb
```
## Example

```golang
package main

import (
	"context"
	"time"

	"github.com/gin-contrib/sessions"
	"github.com/gin-gonic/gin"
	ginsessionmongodb "github.com/vincentinttsh/gin-session-mongodb"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)

func main() {
	r := gin.Default()
	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()
	connect := options.Client().ApplyURI("localhost:27017")
	client, err := mongo.Connect(ctx, connect)
	if err != nil {
		panic(err)
	}
	coll := client.Database("test").Collection("sessions")
	store := ginsessionmongodb.NewStore(coll, 3600, true, []byte("secret"))
	r.Use(sessions.Sessions("mysession", store))

	r.GET("/hello", func(c *gin.Context) {
		session := sessions.Default(c)

		if session.Get("hello") != "world" {
			session.Set("hello", "world")
			session.Save()
		}

		c.JSON(200, gin.H{"hello": session.Get("hello")})
	})
	r.Run(":8000")
}

```
