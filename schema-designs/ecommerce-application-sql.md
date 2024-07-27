- create extensions

```javascript
CREATE EXTENSION IF NOT EXISTS 'uuid-ossp';
```

- create tables
  [text](https://chatgpt.com/c/eb4731e7-369d-4b6f-ad3c-77fdd0486838)

```javascript
CREATE TABLE IF NOT EXISTS users(
    id SERIAL PRIMARY KEY,
    firstName VARCHAR(50) NOT NULL,
    lastName VARCHAR(50) NOT NULL,
    password VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    countryCode VARCHAR(3),
    phoneNumber VARCHAR(15),
    gender CHAR(1),
    city VARCHAR(20),
    state VARCHAR(20),
    createdAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS addresses (
    id SERIAL PRIMARY KEY,
    userId INT REFERENCES users(id),
    firstName VARCHAR(50) NOT NULL,
    lastName VARCHAR(50) NOT NULL,
    postalCode VARCHAR(20) NOT NULL,
    address VARCHAR(255) NOT NULL,
    landMark VARCHAR(255),
    city VARCHAR(100) NOT NULL,
    state VARCHAR(100) NOT NULL,
    country VARCHAR(100) NOT NULL,
    addressType VARCHAR(50), -- e.g., 'shipping', 'billing',
    countryCode VARCHAR(3),
    altPhoneNumber VARCHAR(15),
    createdAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS otpVerification (id SERIAL PRIMARY KEY);
CREATE TABLE IF NOT EXISTS products (id SERIAL PRIMARY KEY);
CREATE TABLE IF NOT EXISTS categories (id SERIAL PRIMARY KEY);
CREATE TABLE IF NOT EXISTS carts (id SERIAL PRIMARY KEY);
CREATE TABLE IF NOT EXISTS orders (id SERIAL PRIMARY KEY);
CREATE TABLE IF NOT EXISTS ratings (id SERIAL PRIMARY KEY);
CREATE TABLE IF NOT EXISTS reviews (id SERIAL PRIMARY KEY);
CREATE TABLE IF NOT EXISTS coupons (id SERIAL PRIMARY KEY);
CREATE TABLE IF NOT EXISTS wishlists (id SERIAL PRIMARY KEY);
```

- delete tables

```javascript
DROP TABLE IF EXISTS users;
DROP TABLE IF EXISTS otpVerification;
DROP TABLE IF EXISTS products;
DROP TABLE IF EXISTS categories;
DROP TABLE IF EXISTS addresses;
DROP TABLE IF EXISTS carts;
DROP TABLE IF EXISTS orders;
DROP TABLE IF EXISTS ratings;
DROP TABLE IF EXISTS reviews;
DROP TABLE IF EXISTS coupons;
DROP TABLE IF EXISTS wishlists;
```

- insert data

```javascript

- Insert User Data

INSERT INTO users (firstName, lastName, password, email, countryCode, phoneNumber, gender, city, state)
VALUES
('John', 'Doe', 'password123', 'john.doe@example.com', '+91', '1234567890', 'M', 'New York', 'NY'),
('Jane', 'Smith', 'securepass', 'jane.smith@example.com', '+91', '2345678901', 'F', 'Toronto', 'ON'),
('Alice', 'Johnson', 'alicepwd', 'alice.johnson@example.com', '+91', '3456789012', 'F', 'London', 'LN'),
('Bob', 'Brown', 'bobbrownpass', 'bob.brown@example.com', '+91', '4567890123', 'M', 'Sydney', 'NSW'),
('Charlie', 'Davis', 'charliedavis', 'charlie.davis@example.com', '+91', '5678901234', 'M', 'Auckland', 'AKL'),
('Diana', 'Evans', 'dianapassword', 'diana.evans@example.com', '+91', '6789012345', 'F', 'Mumbai', 'MH'),
('Edward', 'Garcia', 'edwardpass', 'edward.garcia@example.com', '+91', '7890123456', 'M', 'Madrid', 'MD'),
('Fiona', 'Harris', 'fionapass', 'fiona.harris@example.com', '+91', '8901234567', 'F', 'Berlin', 'BE'),
('George', 'Clark', 'georgepass', 'george.clark@example.com', '+91', '9012345678', 'M', 'Paris', 'PA'),
('Hannah', 'Walker', 'hannahpass', 'hannah.walker@example.com', '+91', '0123456789', 'F', 'Rome', 'RM');

- Insert Address Data

-- Insert demo address data for users with different names
INSERT INTO addresses (userId, firstName, lastName, postalCode, address, landMark, city, state, country, addressType, countryCode, altPhoneNumber)
VALUES
-- Address for userId 1
(1, 'John', 'Doe', '10001', '123 Main St', 'Near Central Park', 'New York', 'NY', 'USA', 'shipping', '1', '1234567890'),
(1, 'Jane', 'Doe', '10001', '456 Elm St', 'Near Times Square', 'New York', 'NY', 'USA', 'billing', '1', '0987654321'),

-- Address for userId 2
(2, 'Robert', 'Smith', 'M5H 2N2', '789 Maple Ave', 'Near CN Tower', 'Toronto', 'ON', 'Canada', 'shipping', '1', '2345678901'),
(2, 'Emily', 'Smith', 'M5H 2N2', '101 Oak St', 'Near Eaton Centre', 'Toronto', 'ON', 'Canada', 'billing', '1', '8765432109'),

-- Address for userId 3
(3, 'Michael', 'Johnson', 'EC1A 1BB', '202 Pine St', 'Near Big Ben', 'London', 'Greater London', 'UK', 'shipping', '44', '3456789012'),
(3, 'Sarah', 'Johnson', 'EC1A 1BB', '303 Cedar St', 'Near Buckingham Palace', 'London', 'Greater London', 'UK', 'billing', '44', '9876543210'),

-- Address for userId 4
(4, 'William', 'Brown', '2000', '404 Spruce St', 'Near Opera House', 'Sydney', 'NSW', 'Australia', 'shipping', '61', '4567890123'),
(4, 'Olivia', 'Brown', '2000', '505 Birch St', 'Near Harbour Bridge', 'Sydney', 'NSW', 'Australia', 'billing', '61', '8765432109'),

-- Address for userId 5
(5, 'James', 'Davis', '1010', '606 Willow St', 'Near Sky Tower', 'Auckland', 'Auckland', 'New Zealand', 'shipping', '64', '5678901234'),
(5, 'Emma', 'Davis', '1010', '707 Fir St', 'Near Auckland Zoo', 'Auckland', 'Auckland', 'New Zealand', 'billing', '64', '7654321098'),

-- Address for userId 6
(6, 'Charles', 'Evans', '400001', '808 Palm St', 'Near Gateway of India', 'Mumbai', 'MH', 'India', 'shipping', '91', '6789012345'),
(6, 'Sophia', 'Evans', '400001', '909 Cedar St', 'Near Marine Drive', 'Mumbai', 'MH', 'India', 'billing', '91', '7654321098'),

-- Address for userId 7
(7, 'Henry', 'Garcia', '28013', '1010 Ash St', 'Near Prado Museum', 'Madrid', 'MD', 'Spain', 'shipping', '34', '7890123456'),
(7, 'Isabella', 'Garcia', '28013', '1111 Cypress St', 'Near Royal Palace', 'Madrid', 'MD', 'Spain', 'billing', '34', '6543210987'),

-- Address for userId 8
(8, 'George', 'Harris', '10178', '1212 Hemlock St', 'Near Brandenburg Gate', 'Berlin', 'BE', 'Germany', 'shipping', '49', '8901234567'),
(8, 'Mia', 'Harris', '10178', '1313 Juniper St', 'Near Berlin Wall', 'Berlin', 'BE', 'Germany', 'billing', '49', '5432109876'),

-- Address for userId 9
(9, 'David', 'Clark', '75001', '1414 Dogwood St', 'Near Eiffel Tower', 'Paris', 'Île-de-France', 'France', 'shipping', '33', '9012345678'),
(9, 'Ava', 'Clark', '75001', '1515 Magnolia St', 'Near Louvre Museum', 'Paris', 'Île-de-France', 'France', 'billing', '33', '4321098765'),

-- Address for userId 10
(10, 'Joseph', 'Walker', '00184', '1616 Poplar St', 'Near Colosseum', 'Rome', 'Genoa', 'Italy', 'shipping', '39', '0123456789'),
(10, 'Amelia', 'Walker', '00184', '1717 Redwood St', 'Near Vatican City', 'Rome', 'Genoa', 'Italy', 'billing', '39', '3210987654');

```
