# Redux-related-files-and-code
import { configureStore } from '@reduxjs/toolkit';
import cartReducer from './cartSlice';
import productsReducer from './productsSlice';

export const store = configureStore({
  reducer: {
    cart: cartReducer,
    products: productsReducer
  }
});

import { createSlice } from '@reduxjs/toolkit';

const initialState = {
  items: [],
  totalQuantity: 0,
  totalAmount: 0
};

const cartSlice = createSlice({
  name: 'cart',
  initialState,
  reducers: {
    addItem(state, action) {
      const newItem = action.payload;
      const existingItem = state.items.find(item => item.id === newItem.id);
      
      if (!existingItem) {
        state.items.push({
          id: newItem.id,
          name: newItem.name,
          price: newItem.price,
          quantity: 1,
          totalPrice: newItem.price,
          image: newItem.image
        });
      } else {
        existingItem.quantity++;
        existingItem.totalPrice += newItem.price;
      }
      
      state.totalQuantity++;
      state.totalAmount += newItem.price;
    },
    removeItem(state, action) {
      const id = action.payload;
      const existingItem = state.items.find(item => item.id === id);
      
      if (existingItem.quantity === 1) {
        state.items = state.items.filter(item => item.id !== id);
      } else {
        existingItem.quantity--;
        existingItem.totalPrice -= existingItem.price;
      }
      
      state.totalQuantity--;
      state.totalAmount -= existingItem.price;
    },
    deleteItem(state, action) {
      const id = action.payload;
      const existingItem = state.items.find(item => item.id === id);
      
      state.items = state.items.filter(item => item.id !== id);
      state.totalQuantity -= existingItem.quantity;
      state.totalAmount -= existingItem.totalPrice;
    }
  }
});

export const cartActions = cartSlice.actions;
export default cartSlice.reducer;

import { createSlice } from '@reduxjs/toolkit';

const initialState = {
  products: [
    {
      id: 'p1',
      name: 'Snake Plant',
      price: 25.99,
      image: 'https://example.com/snake-plant.jpg',
      category: 'Low Maintenance'
    },
    {
      id: 'p2',
      name: 'Monstera Deliciosa',
      price: 35.50,
      image: 'https://example.com/monstera.jpg',
      category: 'Tropical'
    },
    // Add 4 more plants with different categories
  ]
};

const productsSlice = createSlice({
  name: 'products',
  initialState,
  reducers: {}
});

export default productsSlice.reducer;

import { Link } from 'react-router-dom';
import './HomePage.css';

function HomePage() {
  return (
    <div className="home-container">
      <div className="hero-section">
        <h1>GreenThumb</h1>
        <p>Your one-stop shop for beautiful houseplants that bring life to your space.</p>
        <Link to="/products">
          <button className="cta-button">Get Started</button>
        </Link>
      </div>
    </div>
  );
}

export default HomePage;

.home-container {
  background: url('https://example.com/plant-bg.jpg') center/cover no-repeat;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  color: white;
}

.hero-section {
  background-color: rgba(0, 0, 0, 0.6);
  padding: 2rem;
  border-radius: 10px;
}

.cta-button {
  background-color: #4CAF50;
  color: white;
  padding: 12px 24px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
  margin-top: 1rem;
}

.cta-button:hover {
  background-color: #45a049;
}import { useSelector } from 'react-redux';
import ProductCard from '../components/ProductCard';
import PlantCategory from '../components/PlantCategory';

function ProductsPage() {
  const products = useSelector(state => state.products.products);
  
  // Group by category
  const categories = [...new Set(products.map(product => product.category))];
  
  return (
    <div className="products-page">
      <h1>Our Plants</h1>
      {categories.map(category => (
        <PlantCategory 
          key={category}
          category={category}
          products={products.filter(p => p.category === category)}
        />
      ))}
    </div>
  );
}

export default ProductsPage;import ProductCard from './ProductCard';

function PlantCategory({ category, products }) {
  return (
    <div className="category-section">
      <h2>{category}</h2>
      <div className="products-grid">
        {products.map(product => (
          <ProductCard key={product.id} product={product} />
        ))}
      </div>
    </div>
  );
}

export default PlantCategory;

javascript
import { useSelector, useDispatch } from 'react-redux';
import { cartActions } from '../redux/cartSlice';
import CartItem from '../components/CartItem';
import { Link } from 'react-router-dom';

function CartPage() {
  const dispatch = useDispatch();
  const cartItems = useSelector(state => state.cart.items);
  const totalQuantity = useSelector(state => state.cart.totalQuantity);
  const totalAmount = useSelector(state => state.cart.totalAmount);
  
  const increaseQuantityHandler = (id) => {
    dispatch(cartActions.addItem({
      id,
      price: cartItems.find(item => item.id === id).price
    }));
  };
  
  const decreaseQuantityHandler = (id) => {
    dispatch(cartActions.removeItem(id));
  };
  
  const deleteItemHandler = (id) => {
    dispatch(cartActions.deleteItem(id));
  };
  
  return (
    <div className="cart-page">
      <h1>Your Shopping Cart</h1>
      
      {cartItems.length === 0 ? (
        <p>Your cart is empty. <Link to="/products">Continue shopping</Link></p>
      ) : (
        <>
          <div className="cart-items">
            {cartItems.map(item => (
              <CartItem
                key={item.id}
                item={item}
                onIncrease={increaseQuantityHandler}
                onDecrease={decreaseQuantityHandler}
                onDelete={deleteItemHandler}
              />
            ))}
          </div>
          
          <div className="cart-summary">
            <h3>Total Items: {totalQuantity}</h3>
            <h3>Total Amount: ${totalAmount.toFixed(2)}</h3>
            <button className="checkout-btn">Proceed to Checkout (Coming Soon)</button>
            <Link to="/products" className="continue-shopping">Continue Shopping</Link>
          </div>
        </>
      )}
    </div>
  );
}

export default CartPage;
src/components/CartItem.jsx
javascript
function CartItem({ item, onIncrease, onDecrease, onDelete }) {
  return (
    <div className="cart-item">
      <img src={item.image} alt={item.name} />
      <div className="item-details">
        <h3>{item.name}</h3>
        <p>${item.price.toFixed(2)} each</p>
      </div>
      <div className="item-controls">
        <button onClick={() => onDecrease(item.id)}>-</button>
        <span>{item.quantity}</span>
        <button onClick={() => onIncrease(item.id)}>+</button>
        <button 
          onClick={() => onDelete(item.id)}
          className="delete-btn"
        >
          Remove
        </button>
      </div>
      <div className="item-total">
        ${item.totalPrice.toFixed(2)}
      </div>
    </div>
  );
}

export default CartItem;
6. App Setup
src/App.js
javascript
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import Header from './components/Header';
import HomePage from './pages/HomePage';
import ProductsPage from './pages/ProductsPage';
import CartPage from './pages/CartPage';
import './App.css';

function App() {
  return (
    <Router>
      <Header />
      <main>
        <Routes>
          <Route path="/" element={<HomePage />} />
          <Route path="/products" element={<ProductsPage />} />
          <Route path="/cart" element={<CartPage />} />
        </Routes>
      </main>
    </Router>
  );
}

export default App;
src/index.js
javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import { Provider } from 'react-redux';
import { store } from './redux/store';
import App from './App';
import './index.css';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);