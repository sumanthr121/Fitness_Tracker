# Fitness_Tracker
-- Drop and Create the Database
DROP DATABASE IF EXISTS fitness_tracker_db;
CREATE DATABASE fitness_tracker_db;
USE fitness_tracker_db;

-- Create Users Table
CREATE TABLE IF NOT EXISTS Users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    date_joined TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create Activities Table
CREATE TABLE IF NOT EXISTS Activities (
    activity_id INT AUTO_INCREMENT PRIMARY KEY,
    activity_name VARCHAR(100) NOT NULL,
    calories_burned INT NOT NULL
);

-- Create Workouts Table (M:N Relationship with Users and Activities)
CREATE TABLE IF NOT EXISTS Workouts (
    workout_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    activity_id INT,
    workout_date DATE,
    duration INT,  -- Duration of the workout in minutes
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE,
    FOREIGN KEY (activity_id) REFERENCES Activities(activity_id) ON DELETE CASCADE
);

-- Create Diets Table
CREATE TABLE IF NOT EXISTS Diets (
    diet_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    meal VARCHAR(255) NOT NULL,
    calories INT NOT NULL,
    meal_date DATE,
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE
);

-- Create Progress Table
CREATE TABLE IF NOT EXISTS Progress (
    progress_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    weight DECIMAL(5,2),
    height DECIMAL(5,2),
    progress_date DATE,
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE
);

-- Add Named Constraint for Workouts table to avoid duplicate user-activity-date combinations
ALTER TABLE Workouts ADD CONSTRAINT unique_workout UNIQUE (user_id, activity_id, workout_date);

-- Create UserProgressSummary View
CREATE VIEW UserProgressSummary AS
SELECT u.user_id, u.username, 
       p.weight, p.height, p.progress_date, 
       SUM(a.calories_burned) AS total_calories_burned
FROM Users u
JOIN Progress p ON u.user_id = p.user_id
JOIN Workouts w ON u.user_id = w.user_id
JOIN Activities a ON w.activity_id = a.activity_id
GROUP BY u.user_id, u.username, p.weight, p.height, p.progress_date;

-- Create UserDetails View (user, diet, and progress information)
CREATE VIEW UserDetails AS
SELECT u.username, d.meal, d.calories, p.weight, p.height
FROM Users u
JOIN Diets d ON u.user_id = d.user_id
JOIN Progress p ON u.user_id = p.user_id;

-- Create Trigger to update progress after a workout
DELIMITER //
CREATE TRIGGER update_progress_after_workout
AFTER INSERT ON Workouts
FOR EACH ROW
BEGIN
    -- Logic to update progress after a workout (e.g., update weight, etc.)
    -- Assuming weight is updated in the Progress table based on the workout.
    UPDATE Progress 
    SET weight = weight - 0.1 -- Just an example; adjust according to actual logic
    WHERE user_id = NEW.user_id AND progress_date = CURDATE();
END;
//
DELIMITER ;

-- Insert Sample Data into Users Table (10 entries)
INSERT INTO Users (username, email, password) VALUES
("JohnDoe", "john@example.com", "password123"),
("JaneSmith", "jane@example.com", "securepassword"),
("MarkTaylor", "mark@example.com", "123password"),
("AliceBrown", "alice@example.com", "alicepass"),
("BobJohnson", "bob@example.com", "bobpassword"),
("CharlieDavis", "charlie@example.com", "charliepass"),
("DavidMartinez", "david@example.com", "davidpass"),
("EvaLopez", "eva@example.com", "evapassword"),
("FrankGarcia", "frank@example.com", "frankpass"),
("GraceWilson", "grace@example.com", "gracepass");

-- Insert Sample Data into Activities Table (10 entries)
INSERT INTO Activities (activity_name, calories_burned) VALUES
("Running", 300),
("Cycling", 250),
("Swimming", 200),
("Yoga", 150),
("Walking", 100),
("HIIT", 400),
("Pilates", 180),
("Hiking", 350),
("Weight Training", 500),
("Boxing", 350);

-- Insert Sample Data into Workouts Table (10 entries)
INSERT INTO Workouts (user_id, activity_id, workout_date, duration) VALUES
(1, 1, "2025-01-25", 30),
(2, 3, "2025-01-24", 45),
(1, 2, "2025-01-23", 60),
(3, 4, "2025-01-22", 30),
(4, 5, "2025-01-21", 40),
(5, 6, "2025-01-20", 50),
(6, 7, "2025-01-19", 45),
(7, 8, "2025-01-18", 60),
(8, 9, "2025-01-17", 40),
(9, 10, "2025-01-16", 30);

-- Insert Sample Data into Diets Table (10 entries)
INSERT INTO Diets (user_id, meal, calories, meal_date) VALUES
(1, "Breakfast", 500, "2025-01-25"),
(2, "Lunch", 700, "2025-01-24"),
(1, "Dinner", 600, "2025-01-23"),
(3, "Snack", 200, "2025-01-22"),
(4, "Breakfast", 450, "2025-01-21"),
(5, "Lunch", 750, "2025-01-20"),
(6, "Dinner", 650, "2025-01-19"),
(7, "Snack", 300, "2025-01-18"),
(8, "Breakfast", 400, "2025-01-17"),
(9, "Lunch", 800, "2025-01-16");

-- Insert Sample Data into Progress Table (10 entries)
INSERT INTO Progress (user_id, weight, height, progress_date) VALUES
(1, 70.5, 175, "2025-01-25"),
(2, 68.0, 160, "2025-01-24"),
(3, 75.0, 180, "2025-01-23"),
(4, 65.0, 170, "2025-01-22"),
(5, 78.5, 165, "2025-01-21"),
(6, 70.0, 172, "2025-01-20"),
(7, 82.0, 178, "2025-01-19"),
(8, 64.0, 160, "2025-01-18"),
(9, 80.5, 175, "2025-01-17"),
(10, 76.0, 180, "2025-01-16");

-- Sample SELECT Queries

-- View all data in UserProgressSummary
SELECT * FROM UserProgressSummary;

-- View all data in UserDetails
SELECT * FROM UserDetails;

-- View all users
SELECT * FROM Users;

-- View all activities
SELECT * FROM Activities;

-- View all workouts
SELECT * FROM Workouts;

-- View all diets
SELECT * FROM Diets;

-- View all progress
SELECT * FROM Progress;
