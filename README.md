const products = [
  {
    id: 1,
    name: "Bamboo Storage Basket",
    category: "home",
    tag: "Best Seller",
    price: 38,
    image:
      "https://images.unsplash.com/photo-1505693416388-ac5ce068fe85?auto=format&fit=crop&w=900&q=80",
  },
  {
    id: 2,
    name: "Organic Cotton Throw",
    category: "home",
    tag: "New",
    price: 58,
    image:
      "https://images.unsplash.com/photo-1524758631624-e2822e304c36?auto=format&fit=crop&w=900&q=80",
  },
  {
    id: 3,
    name: "Citrus Glow Serum",
    category: "wellness",
    tag: "Popular",
    price: 42,
    image:
      "https://images.unsplash.com/photo-1556228578-8c89e6adf883?auto=format&fit=crop&w=900&q=80",
  },
  {
    id: 4,
    name: "Glass Water Bottle",
    category: "accessories",
    tag: "Eco Pick",
    price: 26,
    image:
      "https://images.unsplash.com/photo-1602143407151-7111542de6e8?auto=format&fit=crop&w=900&q=80",
  },
  {
    id: 5,
    name: "Essential Oil Diffuser",
    category: "wellness",
    tag: "Top Rated",
    price: 64,
    image:
      "https://images.unsplash.com/photo-1515377905703-c4788e51af15?auto=format&fit=crop&w=900&q=80",
  },
  {
    id: 6,
    name: "Woven Tote Bag",
    category: "accessories",
    tag: "Fresh Drop",
    price: 34,
    image:
      "https://images.unsplash.com/photo-1525966222134-fcfa99b8ae77?auto=format&fit=crop&w=900&q=80",
  },
  {
    id: 7,
    name: "Plant Pot Set",
    category: "home",
    tag: "Limited",
    price: 48,
    image:
      "https://images.unsplash.com/photo-1466692476868-aef1dfb1e735?auto=format&fit=crop&w=900&q=80",
  },
  {
    id: 8,
    name: "Natural Bamboo Brush",
    category: "wellness",
    tag: "Daily Ritual",
    price: 18,
    image:
      "https://images.unsplash.com/photo-1522335789203-aabd1fc54bc9?auto=format&fit=crop&w=900&q=80",
  },
];

const productGrid = document.getElementById("productGrid");
const searchInput = document.getElementById("searchInput");
const filterButtons = document.querySelectorAll(".filter-chip");
const cartButton = document.querySelector(".cart-button");
const cartDrawer = document.querySelector(".cart-drawer");
const closeCartButton = document.querySelector(".close-cart");
const overlay = document.querySelector(".overlay");
const cartItemsContainer = document.getElementById("cartItems");
const cartCount = document.getElementById("cart-count");
const subtotalEl = document.getElementById("subtotal");
const shippingEl = document.getElementById("shipping");
const totalEl = document.getElementById("total");

let activeFilter = "all";
let cart = [];

function formatPrice(value) {
  return new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
  }).format(value);
}

function getFilteredProducts() {
  const query = searchInput.value.trim().toLowerCase();

  return products.filter((product) => {
    const matchesFilter = activeFilter === "all" || product.category === activeFilter;
    const matchesSearch =
      !query ||
      product.name.toLowerCase().includes(query) ||
      product.category.toLowerCase().includes(query) ||
      product.tag.toLowerCase().includes(query);

    return matchesFilter && matchesSearch;
  });
}

function renderProducts() {
  const filteredProducts = getFilteredProducts();

  if (filteredProducts.length === 0) {
    productGrid.innerHTML = `
      <div class="empty-cart" style="grid-column: 1 / -1;">
        No products match your search. Try another keyword or filter.
      </div>
    `;
    return;
  }

  productGrid.innerHTML = filteredProducts
    .map(
      (product) => `
        <article class="product-card" aria-label="${product.name}">
          <div class="product-media">
            <img src="${product.image}" alt="${product.name}" />
            <span class="product-badge">${product.tag}</span>
          </div>
          <div class="product-info">
            <div class="product-meta">
              <span>${product.category}</span>
              <span>4.9 ★</span>
            </div>
            <h3>${product.name}</h3>
            <div class="product-footer">
              <div class="price">${formatPrice(product.price)}</div>
              <button class="add-cart" data-id="${product.id}">Add to cart</button>
            </div>
          </div>
        </article>
      `
    )
    .join("");

  document.querySelectorAll(".add-cart").forEach((button) => {
    button.addEventListener("click", () => addToCart(Number(button.dataset.id)));
  });
}

function addToCart(productId) {
  const product = products.find((item) => item.id === productId);
  if (!product) return;

  const existingItem = cart.find((item) => item.id === productId);

  if (existingItem) {
    existingItem.quantity += 1;
  } else {
    cart.push({ ...product, quantity: 1 });
  }

  updateCart();
  openCart();
}

function updateCart() {
  const totalItems = cart.reduce((sum, item) => sum + item.quantity, 0);
  cartCount.textContent = String(totalItems);

  if (cart.length === 0) {
    cartItemsContainer.innerHTML = `
      <div class="empty-cart">
        Your cart is empty. Add a few planet-friendly essentials to get started.
      </div>
    `;
    subtotalEl.textContent = formatPrice(0);
    shippingEl.textContent = formatPrice(0);
    totalEl.textContent = formatPrice(0);
    return;
  }

  const subtotal = cart.reduce((sum, item) => sum + item.price * item.quantity, 0);
  const shipping = subtotal > 70 ? 0 : 9.99;
  const total = subtotal + shipping;

  cartItemsContainer.innerHTML = cart
    .map(
      (item) => `
        <div class="cart-item">
          <img src="${item.image}" alt="${item.name}" />
          <div class="item-details">
            <h4>${item.name}</h4>
            <p>${formatPrice(item.price)} each</p>
          </div>
          <div class="item-pricing">
            <div class="qty-controls" aria-label="Quantity controls for ${item.name}">
              <button type="button" data-action="decrease" data-id="${item.id}">-</button>
              <span>${item.quantity}</span>
              <button type="button" data-action="increase" data-id="${item.id}">+</button>
            </div>
            <button class="remove-item" data-action="remove" data-id="${item.id}">Remove</button>
          </div>
        </div>
      `
    )
    .join("");

  subtotalEl.textContent = formatPrice(subtotal);
  shippingEl.textContent = formatPrice(shipping);
  totalEl.textContent = formatPrice(total);

  cartItemsContainer.querySelectorAll("button[data-action]").forEach((button) => {
    button.addEventListener("click", () => updateCartItem(Number(button.dataset.id), button.dataset.action));
  });
}

function updateCartItem(productId, action) {
  const item = cart.find((entry) => entry.id === productId);
  if (!item) return;

  if (action === "increase") {
    item.quantity += 1;
  }

  if (action === "decrease") {
    item.quantity -= 1;
    if (item.quantity <= 0) {
      cart = cart.filter((entry) => entry.id !== productId);
    }
  }

  if (action === "remove") {
    cart = cart.filter((entry) => entry.id !== productId);
  }

  updateCart();
}

function openCart() {
  cartDrawer.classList.add("open");
  overlay.classList.add("show");
}

function closeCart() {
  cartDrawer.classList.remove("open");
  overlay.classList.remove("show");
}

searchInput.addEventListener("input", renderProducts);

filterButtons.forEach((button) => {
  button.addEventListener("click", () => {
    activeFilter = button.dataset.filter;
    filterButtons.forEach((chip) => chip.classList.toggle("active", chip === button));
    renderProducts();
  });
});

cartButton.addEventListener("click", openCart);
closeCartButton.addEventListener("click", closeCart);
overlay.addEventListener("click", closeCart);

document.querySelector(".cart-checkout").addEventListener("click", () => {
  alert("Checkout started! Your eco-friendly order is ready to be processed.");
});

document.querySelector(".newsletter-form").addEventListener("submit", (event) => {
  event.preventDefault();
  const emailInput = event.currentTarget.querySelector("input");
  if (emailInput.value.trim()) {
    alert("Thanks for joining Ecoomars! You’re on the list for our next eco drop.");
    emailInput.value = "";
  }
});

renderProducts();
updateCart();

