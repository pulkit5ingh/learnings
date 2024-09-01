# E-commerce Website Schema Design

## 1. Users Collection

- **Fields:**

  - `_id`: ObjectId (Primary Key)
  - `username`: String (Unique)
  - `email`: String (Unique)
  - `password`: String
  - `profile`:
    - `firstName`: String
    - `lastName`: String
    - `address`: Embedded Document (street, city, state, postalCode, country)
  - `orders`: Array of ObjectId (References `Orders._id`)
  - `cart`: Array of ObjectId (References `Cart._id`)
  - `wishlist`: Array of ObjectId (References `Products._id`)
  - `createdAt`: Date
  - `updatedAt`: Date

- **Relationships:**
  - 1-to-many with `Orders`: A user can have multiple orders.
  - 1-to-many with `Cart`: A user can have multiple items in the cart.
  - many-to-many with `Products` via Wishlist: A user can have multiple products in their wishlist, and a product can be in multiple users' wishlists.

## 2. Products Collection

- **Fields:**

  - `_id`: ObjectId (Primary Key)
  - `name`: String
  - `description`: String
  - `price`: Decimal
  - `stock`: Number
  - `category`: ObjectId (References `Categories._id`)
  - `images`: Array of Strings (URLs)
  - `attributes`:
    - `color`: String
    - `size`: String
    - `weight`: Number
  - `reviews`: Array of ObjectId (References `Reviews._id`)
  - `createdAt`: Date
  - `updatedAt`: Date

- **Relationships:**
  - 1-to-many with `Reviews`: A product can have multiple reviews.
  - many-to-one with `Categories`: A product belongs to one category.
  - many-to-many with `Users` via Wishlist: A product can be in multiple users' wishlists.

## 3. Categories Collection

- **Fields:**

  - `_id`: ObjectId (Primary Key)
  - `name`: String (Unique)
  - `description`: String
  - `parentCategory`: ObjectId (References `Categories._id`, optional, for sub-categories)
  - `createdAt`: Date
  - `updatedAt`: Date

- **Relationships:**
  - 1-to-many with `Products`: A category can have multiple products.
  - Self-referencing 1-to-many for sub-categories.

## 4. Orders Collection

- **Fields:**

  - `_id`: ObjectId (Primary Key)
  - `userId`: ObjectId (References `Users._id`)
  - `items`: Array of Embedded Documents
    - `productId`: ObjectId (References `Products._id`)
    - `quantity`: Number
    - `price`: Decimal (Snapshot of the price at the time of order)
  - `totalPrice`: Decimal
  - `status`: String (e.g., "Pending", "Shipped", "Delivered", "Cancelled")
  - `shippingAddress`: Embedded Document (Copy from `Users.address` or a new address)
  - `paymentMethod`: String (e.g., "Credit Card", "PayPal")
  - `createdAt`: Date
  - `updatedAt`: Date

- **Relationships:**
  - 1-to-many with `Users`: A user can place multiple orders.
  - many-to-one with `Products`: An order can contain multiple products.

## 5. Cart Collection

- **Fields:**

  - `_id`: ObjectId (Primary Key)
  - `userId`: ObjectId (References `Users._id`)
  - `items`: Array of Embedded Documents
    - `productId`: ObjectId (References `Products._id`)
    - `quantity`: Number
  - `createdAt`: Date
  - `updatedAt`: Date

- **Relationships:**
  - 1-to-1 with `Users`: Each user has one cart.
  - many-to-one with `Products`: A cart can contain multiple products.

## 6. Reviews Collection

- **Fields:**

  - `_id`: ObjectId (Primary Key)
  - `productId`: ObjectId (References `Products._id`)
  - `userId`: ObjectId (References `Users._id`)
  - `rating`: Number (1-5)
  - `comment`: String
  - `createdAt`: Date
  - `updatedAt`: Date

- **Relationships:**
  - 1-to-many with `Products`: A product can have multiple reviews.
  - 1-to-many with `Users`: A user can write multiple reviews.

## 7. Payments Collection

- **Fields:**

  - `_id`: ObjectId (Primary Key)
  - `orderId`: ObjectId (References `Orders._id`)
  - `userId`: ObjectId (References `Users._id`)
  - `paymentMethod`: String (e.g., "Credit Card", "PayPal")
  - `amount`: Decimal
  - `status`: String (e.g., "Completed", "Pending", "Failed")
  - `transactionId`: String (Unique identifier from payment gateway)
  - `createdAt`: Date
  - `updatedAt`: Date

- **Relationships:**
  - 1-to-1 with `Orders`: Each order has one payment record.
  - 1-to-many with `Users`: A user can have multiple payment records.

## 8. Shipping Collection

- **Fields:**

  - `_id`: ObjectId (Primary Key)
  - `orderId`: ObjectId (References `Orders._id`)
  - `trackingNumber`: String (Unique)
  - `carrier`: String (e.g., "UPS", "FedEx")
  - `status`: String (e.g., "In Transit", "Delivered")
  - `estimatedDelivery`: Date
  - `actualDelivery`: Date (optional)
  - `createdAt`: Date
  - `updatedAt`: Date

- **Relationships:**
  - 1-to-1 with `Orders`: Each order has one shipping record.

## 9. Wishlists Collection

- **Fields:**

  - `_id`: ObjectId (Primary Key)
  - `userId`: ObjectId (References `Users._id`)
  - `products`: Array of ObjectId (References `Products._id`)
  - `createdAt`: Date
  - `updatedAt`: Date

- **Relationships:**
  - 1-to-1 with `Users`: Each user can have one wishlist.
  - many-to-many with `Products`: A wishlist can contain multiple products, and a product can be in multiple wishlists.

# E-commerce Website Api designs

- **Users apis:**

```js
app.get('/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id).populate(
    'orders cart wishlist'
  );
  res.status(200).json(user);
});

app.put('/users/:id', async (req, res) => {
  const updatedUser = await User.findByIdAndUpdate(req.params.id, req.body, {
    new: true,
  });
  res.status(200).json(updatedUser);
});

app.delete('/users/:id', async (req, res) => {
  await User.findByIdAndDelete(req.params.id);
  res.status(204).send();
});
```

- **Products apis:**

```js
app.post('/products', async (req, res) => {
  const newProduct = new Product(req.body);
  await newProduct.save();
  res.status(201).json(newProduct);
});

app.get('/products/:id', async (req, res) => {
  const product = await Product.findById(req.params.id).populate(
    'reviews category'
  );
  res.status(200).json(product);
});

app.put('/products/:id', async (req, res) => {
  const updatedProduct = await Product.findByIdAndUpdate(
    req.params.id,
    req.body,
    { new: true }
  );
  res.status(200).json(updatedProduct);
});
```

- **Categories apis:**

```js
app.get('/reviews/product/:productId', async (req, res) => {
  const reviews = await Review.find({
    productId: req.params.productId,
  }).populate('userId');
  res.status(200).json(reviews);
});
```
