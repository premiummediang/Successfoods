# Successfoods
Success Food Limited
// backend/server.js
require('dotenv').config();
const express = require('express');
const helmet = require('helmet');
const { body, validationResult } = require('express-validator');
const { Sequelize, DataTypes } = require('sequelize');

const app = express();
app.use(helmet());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Database configuration
const sequelize = new Sequelize(
  process.env.DB_NAME,
  process.env.DB_USER,
  process.env.DB_PASSWORD,
  {
    host: process.env.DB_HOST,
    dialect: 'postgres',
    logging: false,
    pool: {
      max: 10,
      min: 0,
      acquire: 30000,
      idle: 10000
    }
  }
);

// Product Model
const Product = sequelize.define('Product', {
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true
  },
  name: {
    type: DataTypes.STRING(100),
    allowNull: false
  },
  description: {
    type: DataTypes.TEXT,
    allowNull: false
  },
  price: {
    type: DataTypes.DECIMAL(10, 2),
    allowNull: false
  },
  sku: {
    type: DataTypes.STRING(50),
    unique: true
  },
  stock: {
    type: DataTypes.INTEGER,
    defaultValue: 0
  },
  imageUrl: {
    type: DataTypes.STRING,
    validate: {
      isUrl: true
    }
  }
}, {
  indexes: [
    {
      fields: ['name'],
      using: 'BTREE'
    },
    {
      fields: ['description'],
      using: 'GIN',
      operator: 'gin_trgm_ops'
    }
  ]
});

// Product API Routes
app.get('/api/products', async (req, res) => {
  try {
    const page = parseInt(req.query.page) || 1;
    const limit = parseInt(req.query.limit) || 50;
    const offset = (page - 1) * limit;

    const { count, rows: products } = await Product.findAndCountAll({
      limit,
      offset,
      order: [['id', 'ASC']]
    });

    res.json({
      totalProducts: count,
      totalPages: Math.ceil(count / limit),
      currentPage: page,
      products
    });
  } catch (error) {
    res.status(500).json({ error: 'Server error' });
  }
});

app.post('/api/products', 
  [
    body('name').trim().isLength({ min: 3, max: 100 }).escape(),
    body('description').trim().isLength({ min: 10 }).escape(),
    body('price').isFloat({ min: 0.01 }),
    body('sku').isAlphanumeric().trim(),
    body('stock').isInt({ min: 0 })
  ],
  async (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }

    try {
      const product = await Product.create(req.body);
      res.status(201).json(product);
    } catch (error) {
      res.status(500).json({ error: 'Product creation failed' });
    }
  }
);

// Error handling middleware
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Internal server error' });
});

// Database connection and server start
sequelize.authenticate()
  .then(() => {
    console.log('Database connected');
    return sequelize.sync({ alter: true });
  })
  .then(() => {
    app.listen(process.env.PORT || 3000, () => {
      console.log(`Server running on port ${process.env.PORT || 3000}`);
    });
  })
  .catch(error => {
    console.error('Database connection failed:', error);
    process.exit(1);
  });
// frontend/src/ProductList.js (React example)
import React, { useState, useEffect } from 'react';
import axios from 'axios';

const ProductList = () => {
  const [products, setProducts] = useState([]);
  const [currentPage, setCurrentPage] = useState(1);
  const [totalPages, setTotalPages] = useState(1);

  useEffect(() => {
    const fetchProducts = async () => {
      try {
        const response = await axios.get(`/api/products?page=${currentPage}`);
        setProducts(response.data.products);
        setTotalPages(response.data.totalPages);
      } catch (error) {
        console.error('Error fetching products:', error);
      }
    };

    fetchProducts();
  }, [currentPage]);

  return (
    <div className="product-list">
      <h1>Products</h1>
      <div className="pagination">
        <button 
          onClick={() => setCurrentPage(p => Math.max(1, p - 1))}
          disabled={currentPage === 1}
        >
          Previous
        </button>
        <span>Page {currentPage} of {totalPages}</span>
        <button
          onClick={() => setCurrentPage(p => Math.min(totalPages, p + 1))}
          disabled={currentPage === totalPages}
        >
          Next
        </button>
      </div>
      <div className="products-grid">
        {products.map(product => (
          <div key={product.id} className="product-card">
            <h3>{product.name}</h3>
            <p>{product.description}</p>
            <p>Price: ${product.price}</p>
            <p>Stock: {product.stock}</p>
          </div>
        ))}
      </div>
    </div>
  );
};

export default ProductList;
