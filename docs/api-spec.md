# API Specification

University Transportation System

This document describes the REST API and WebSocket endpoints used in the system.

---

## Authentication

### Register User

POST /auth/register

Body:
{
  "name": "string",
  "email": "string",
  "password": "string"
}

---

### Login

POST /auth/login

Body:
{
  "email": "string",
  "password": "string"
}

---

## Buses

### Get All Buses

GET /buses

---

### Get Bus Details

GET /buses/{busId}

---

## Trips

### Get Trips

GET /trips

---

### Create Trip

POST /trips

Body:
{
  "bus_id": "number",
  "driver_id": "number",
  "route": "string",
  "departure_time": "datetime"
}

---

## Location Tracking

### Update Bus Location (Driver)

POST /location/update

Body:
{
  "bus_id": "number",
  "latitude": "number",
  "longitude": "number",
  "timestamp": "datetime"
}

---

### Get Bus Location

GET /location/{busId}

---

## WebSocket

Endpoint:
/ws/location-stream

Used for real-time GPS updates from drivers to students.
