<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Tienda Interna EMCALA/SURALNOR</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet" />
    <style>
        body { font-family: 'Inter', sans-serif; }
        .btn-admin {
            @apply bg-blue-500 text-white font-bold py-3 px-6 rounded-lg hover:bg-blue-600 transition;
        }
        .modal {
            @apply fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full flex items-center justify-center z-50;
        }
        .modal-content {
            @apply bg-white rounded-lg shadow-xl p-6 sm:p-8 m-4 max-w-lg w-full;
        }
    </style>
</head>
<body class="bg-gray-100 min-h-screen flex items-center justify-center px-4">
    <main id="app" class="container max-w-3xl bg-white shadow-xl rounded-2xl p-8 text-center space-y-8">
        <header>
            <h1 class="text-3xl sm:text-4xl font-bold text-gray-800">Tienda Interna EMCALA/SURALNOR</h1>
            <p class="mt-2 text-gray-600 text-base sm:text-lg">Panel de administración de pedidos de merchandising</p>
        </header>

        <!-- Login -->
        <section id="login-section" class="w-full">
            <div class="bg-gray-50 p-6 rounded-xl shadow-inner text-left space-y-4">
                <h2 class="text-xl font-bold text-gray-700">Acceso</h2>
                <form id="login-form" class="space-y-4">
                    <input type="text" id="dni-input" autocomplete="username" aria-label="DNI" placeholder="Usuario (DNI)" required class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none" />
                    <input type="password" id="password-input" autocomplete="current-password" aria-label="Contraseña" placeholder="Contraseña" required class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none" />
                    <button type="submit" id="login-btn" class="w-full bg-blue-600 text-white font-bold py-3 rounded-lg hover:bg-blue-700 transition">Acceder</button>
                    <p id="login-error" class="hidden text-sm text-red-500">Usuario o contraseña incorrectos.</p>
                </form>
            </div>
        </section>

        <!-- Admin Panel -->
        <section id="admin-panel" class="hidden w-full">
            <h2 class="text-2xl font-semibold text-gray-700 mb-4">Panel de Administración</h2>
            <div class="flex flex-col gap-4">
                <button id="admin-users-btn" class="btn-admin">Gestionar Usuarios</button>
                <button id="admin-products-btn" class="btn-admin">Gestionar Productos</button>
                <button id="admin-orders-btn" class="btn-admin">Resumen de Pedidos</button>
                <button id="go-to-shop-btn" class="btn-admin bg-purple-600 hover:bg-purple-700">Ir a la Tienda</button>
            </div>
            <div id="admin-content-section" class="mt-8"></div>
        </section>

        <!-- Tienda Usuario -->
        <section id="user-shop" class="hidden w-full">
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-2xl font-semibold text-gray-700">
                    Hola, <span id="user-name-display" class="font-bold"></span>!
                </h2>
                <button id="view-cart-btn" class="bg-green-500 text-white font-bold py-2 px-4 rounded-lg hover:bg-green-600 transition">
                    Ver Carrito (<span id="cart-count">0</span>)
                </button>
            </div>
            
            <div id="shop-message" class="hidden text-green-600 bg-green-100 p-3 rounded-lg font-medium mb-4"></div>

            <div id="category-list" class="flex flex-wrap justify-center gap-4"></div>
            <div id="product-list" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6 mt-6 hidden"></div>
            <button id="back-to-categories-btn" class="mt-4 hidden bg-gray-500 text-white py-2 px-4 rounded-lg hover:bg-gray-600 transition">
                Volver a Categorías
            </button>
            <!-- 🔹 Este botón solo lo ve el ADMIN -->
            <button id="back-to-admin-btn" class="mt-4 hidden bg-purple-600 text-white py-2 px-4 rounded-lg hover:bg-purple-700 transition">
                🔙 Volver al Panel de Administración
            </button>
        </section>
        
        <!-- Carrito de Compras (Modal) -->
        <div id="cart-modal" class="modal hidden">
            <div class="modal-content">
                <h2 class="text-2xl font-bold mb-4">Tu Carrito</h2>
                <ul id="cart-items-list" class="space-y-4 max-h-80 overflow-y-auto mb-4">
                    <!-- Los productos del carrito se renderizarán aquí -->
                </ul>
                <div class="text-right font-bold text-xl mb-4">
                    Total: <span id="cart-total-price">0.00</span>
                </div>
                <div class="flex justify-end space-x-2">
                    <button id="close-cart-btn" class="bg-gray-400 text-white py-2 px-4 rounded-lg hover:bg-gray-500 transition">Cerrar</button>
                    <button id="clear-cart-btn" class="bg-red-500 text-white py-2 px-4 rounded-lg hover:bg-red-600 transition">Vaciar Carrito</button>
                    <button id="checkout-btn" class="bg-green-600 text-white py-2 px-4 rounded-lg hover:bg-green-700 transition">Realizar Pedido</button>
                </div>
            </div>
        </div>

        <!-- Footer Info -->
        <footer>
            <p class="text-sm text-gray-400">ID de Usuario: <span id="user-id-display" class="font-mono"></span></p>
        </footer>
    </main>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, onSnapshot, query, getDocs, setDoc, doc, where } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        setLogLevel('debug');

        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        let firebaseConfig = null;
        try {
            if (typeof __firebase_config !== 'undefined' && __firebase_config) {
                firebaseConfig = JSON.parse(__firebase_config);
            }
        } catch (error) {
            console.error("Error al analizar la configuración de Firebase:", error);
        }
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        // Elements
        const loginSection = document.getElementById('login-section');
        const loginForm = document.getElementById('login-form');
        const loginBtn = document.getElementById('login-btn');
        const dniInput = document.getElementById('dni-input');
        const passwordInput = document.getElementById('password-input');
        const loginError = document.getElementById('login-error');
        const adminPanel = document.getElementById('admin-panel');
        const userShop = document.getElementById('user-shop');
        const userIdDisplay = document.getElementById('user-id-display');
        const userNameDisplay = document.getElementById('user-name-display');
        const categoryList = document.getElementById('category-list');
        const productList = document.getElementById('product-list');
        const shopMessage = document.getElementById('shop-message');
        const adminContentSection = document.getElementById('admin-content-section');
        const backToCategoriesBtn = document.getElementById('back-to-categories-btn');
        const backToAdminBtn = document.getElementById('back-to-admin-btn');
        const goToShopBtn = document.getElementById('go-to-shop-btn');
        const cartModal = document.getElementById('cart-modal');
        const viewCartBtn = document.getElementById('view-cart-btn');
        const closeCartBtn = document.getElementById('close-cart-btn');
        const clearCartBtn = document.getElementById('clear-cart-btn');
        const checkoutBtn = document.getElementById('checkout-btn');
        const cartItemsList = document.getElementById('cart-items-list');
        const cartCount = document.getElementById('cart-count');
        const cartTotalPrice = document.getElementById('cart-total-price');

        let app, auth, db, userId;
        const ADMIN_DNI = "42236799";
        const ADMIN_PASSWORD = "42236799";
        let cart = []; // El carrito de compras

        // Datos iniciales tal como estaban en tu archivo original
        const userDataString = `USUARIO;CONTRASEÑA;NOMBRE
2222222;2222222;JUAN MARTIN
3333333;3333333;SAN JUAN
4444444;4444444;CARLOS RIOS
${ADMIN_DNI};${ADMIN_PASSWORD};Administrador
`;
        const stockDataString = `nombre; Precio ;STOCK;CATEGORIA
leche liviana 2%apostoles x12 1l 90bxp;$ 16.139,28;100;MARKETPLACE
leche liviana 2%apostoles x12 1l 90bxp;$ 16.139,28;50;GASEOSAS
3p-leche liviana 2% apostoles x12 1l;$ 12.765,16;30;CERVEZAS
3p-leche liviana 2% apostoles x12 1l;$ 12.765,16;14;MARKETPLACE
cerveza artesanal;$ 5.000,00;200;CERVEZAS
gaseosa cola;$ 3.500,00;500;GASEOSAS
merch - remera;$ 25.000,00;50;MERCH
merch - gorra;$ 15.000,00;75;MERCH
`;
        // pedidos históricos (csv) que venías cargando
        const orderDataString = `Timestamp;Nombre y apellido;  Área / Sector  ;Telefono/Mail;   Two-pack Vasos Bud 2023 – $14.000  ;COMBO MERCH 1 (SIN STOCK);COMBO MERCH 2 (COPA STELLA + CHOPP QUILMES GRATIS)$19.800;  Two-pack Copas Stella Artois Edicion vintage – $21.268;Copa stella original x2 $19.800;  Tabla Ilustrada Patagonia – $25.413;BAGG DE CORONA $26.300;  One-pack Quilmes Chopp – $8.000  ;  Coolbag Jägermeister – $26.000  ;  Two-pack Vasos Andes Origen – $15.000;ECO VASO BOCA(plastico) $800;ECO VASO RIVER(plastico)$800; 
8/15/2025 10:45:45;Ferreyra Leonardo ;Ventas;1164189475;;;;;;;;;1;;;;\r\n8/15/2025 10:46:24;Cristian vazquez ;Chofer ;1148703637;;;;;;;;;;;5;5;\r\n8/15/2025 10:48:52;Sergio galvan;Repositores;1123635134;1;1;;1;;;;;1;;;2;\r\n`;

        let usersData = [];
        let stockData = [];
        let ordersData = []; // Aquí guardaremos todos los pedidos (CSV histórico + locales)

        function parseCSV(csvString) {
            const lines = csvString.trim().split('\n').filter(l=>l.trim() !== '');
            if (lines.length < 1) return [];
            const headers = lines[0].split(';').map(h => h.trim());
            return lines.slice(1).map(line => {
                const values = line.split(';').map(v => v.trim());
                const obj = {};
                headers.forEach((header, i) => {
                    obj[header] = values[i] || '';
                });
                return obj;
            });
        }

        // Parse initial data
        usersData = parseCSV(userDataString);
        stockData = parseCSV(stockDataString);
        ordersData = parseCSV(orderDataString);

        // -----------------------
        // Local orders persistence
        // -----------------------
        const LOCAL_ORDERS_KEY = 'tienda_local_orders_v1';

        function loadLocalOrders() {
            try {
                const raw = localStorage.getItem(LOCAL_ORDERS_KEY);
                if (!raw) return [];
                const parsed = JSON.parse(raw);
                if (!Array.isArray(parsed)) return [];
                return parsed;
            } catch (e) {
                console.warn("Error leyendo pedidos locales:", e);
                return [];
            }
        }

        function saveLocalOrders(arr) {
            try {
                localStorage.setItem(LOCAL_ORDERS_KEY, JSON.stringify(arr));
            } catch (e) {
                console.error("No se pudo guardar pedidos locales:", e);
            }
        }

        // merge local orders into ordersData (normalize to object with keys similar al CSV)
        function mergeLocalOrdersIntoOrdersData() {
            const local = loadLocalOrders();
            local.forEach(o => {
                // convert each local order into an object where keys are: Timestamp;Nombre;Contacto;Product (we'll flatten products into several rows when rendering)
                // For consistency with renderOrdersView we will store each local order as:
                // { Timestamp, 'Nombre y apellido', 'Contacto', 'items': [{name, qty, price}], total }
                // We'll push the raw object and handle rendering specially.
                ordersData.push(o);
            });
        }

        // -----------------------
        // Firebase init (opcional)
        // -----------------------
        async function initializeFirebase() {
            try {
                if (firebaseConfig && Object.keys(firebaseConfig).length > 0) {
                    app = initializeApp(firebaseConfig);
                    auth = getAuth(app);
                    db = getFirestore(app);
                    
                    onAuthStateChanged(auth, async (user) => {
                        if (user) {
                            userId = user.uid;
                            userIdDisplay.textContent = userId;
                        }
                    });

                    if (initialAuthToken) {
                        await signInWithCustomToken(auth, initialAuthToken);
                    } else {
                        await signInAnonymously(auth);
                    }
                } else {
                    console.log("Aviso: La base de datos no está conectada. Se usará solo el modo local.");
                }
            } catch (error) {
                console.error("Error al iniciar Firebase:", error);
            }
        }

        // -----------------------
        // Login / Render flows
        // -----------------------
        async function handleLogin(e) {
            e.preventDefault();
            const dni = dniInput.value.trim();
            const password = passwordInput.value.trim();
            loginBtn.textContent = 'Verificando...';
            loginBtn.disabled = true;
            loginError.classList.add('hidden');

            const user = usersData.find(u => u.USUARIO === dni && u.CONTRASEÑA === password);

            if (user) {
                userNameDisplay.textContent = user.NOMBRE;
                loginSection.classList.add('hidden');

                if (user.USUARIO === ADMIN_DNI) {
                    adminPanel.classList.remove('hidden');
                    renderAdminDashboard();
                    backToAdminBtn.classList.remove('hidden');
                } else {
                    userShop.classList.remove('hidden');
                    renderUserShop();
                }
            } else {
                loginError.classList.remove('hidden');
            }
            loginBtn.textContent = 'Acceder';
            loginBtn.disabled = false;
        }

        function renderAdminDashboard() {
            mergeLocalOrdersIntoOrdersData(); // aseguramos que los pedidos locales estén disponibles
            renderOrdersView();
        }

        function renderUsersView() {
            adminContentSection.innerHTML = `
                <h3 class="text-xl font-semibold mb-4 text-gray-700">Gestionar Usuarios</h3>
                <div class="bg-gray-50 p-6 rounded-xl shadow-inner mb-6 space-y-4 text-left">
                    <h4 class="font-semibold text-gray-700">Cargar usuarios desde archivo CSV</h4>
                    <p class="text-sm text-gray-500">
                        Sube un archivo CSV con las columnas: <strong>USUARIO;CONTRASEÑA;NOMBRE</strong>.
                    </p>
                    <input type="file" id="users-csv-file-input" accept=".csv" class="w-full text-sm text-gray-500
                        file:mr-4 file:py-2 file:px-4
                        file:rounded-full file:border-0
                        file:text-sm file:font-semibold
                        file:bg-blue-50 file:text-blue-700
                        hover:file:bg-blue-100
                    "/>
                    <button id="upload-users-btn" class="w-full bg-blue-600 text-white font-bold py-2 rounded-lg hover:bg-blue-700 transition">Cargar archivo</button>
                    <div id="users-upload-message" class="mt-2 text-sm"></div>
                </div>
                <div class="overflow-x-auto">
                    <table class="min-w-full bg-white border border-gray-200 rounded-lg">
                        <thead class="bg-gray-100">
                            <tr>
                                <th class="py-2 px-4 border-b">DNI</th>
                                <th class="py-2 px-4 border-b">Nombre</th>
                            </tr>
                        </thead>
                        <tbody id="users-table-body"></tbody>
                    </table>
                </div>
            `;
            const tableBody = document.getElementById("users-table-body");
            usersData.forEach(user => {
                const row = document.createElement('tr');
                row.innerHTML = `<td class="py-2 px-4 border-b">${user.USUARIO}</td><td class="py-2 px-4 border-b">${user.NOMBRE}</td>`;
                tableBody.appendChild(row);
            });

            // Lógica de subida de archivo para usuarios
            const usersCsvInput = document.getElementById('users-csv-file-input');
            const uploadBtn = document.getElementById('upload-users-btn');
            const uploadMessage = document.getElementById('users-upload-message');

            uploadBtn.addEventListener('click', () => {
                const file = usersCsvInput.files[0];
                if (!file) {
                    uploadMessage.textContent = "Por favor, selecciona un archivo CSV.";
                    uploadMessage.className = "mt-2 text-sm text-red-600";
                    return;
                }

                const reader = new FileReader();
                reader.onload = (event) => {
                    try {
                        const newUsersData = parseCSV(event.target.result);
                        if (newUsersData.length > 0 && 'USUARIO' in newUsersData[0] && 'CONTRASEÑA' in newUsersData[0] && 'NOMBRE' in newUsersData[0]) {
                            usersData = newUsersData;
                            renderUsersView(); // Volver a renderizar la tabla
                            uploadMessage.textContent = "Archivo de usuarios cargado y actualizado correctamente.";
                            uploadMessage.className = "mt-2 text-sm text-green-600";
                        } else {
                            uploadMessage.textContent = "Formato de archivo incorrecto. Asegúrate de que las columnas son: 'USUARIO;CONTRASEÑA;NOMBRE'";
                            uploadMessage.className = "mt-2 text-sm text-red-600";
                        }
                    } catch (e) {
                        uploadMessage.textContent = "Error al procesar el archivo. Verifica el formato.";
                        uploadMessage.className = "mt-2 text-sm text-red-600";
                    }
                };
                reader.onerror = () => {
                    uploadMessage.textContent = "Error al leer el archivo.";
                    uploadMessage.className = "mt-2 text-sm text-red-600";
                };
                reader.readAsText(file);
            });
        }

        function renderProductsView() {
            adminContentSection.innerHTML = `
                <h3 class="text-xl font-semibold mb-4 text-gray-700">Gestionar Productos</h3>
                
                <div class="bg-gray-50 p-6 rounded-xl shadow-inner mb-6 space-y-4 text-left">
                    <h4 class="font-semibold text-gray-700">Cargar productos desde archivo CSV</h4>
                    <p class="text-sm text-gray-500">
                        Sube un archivo CSV con las columnas: <strong>nombre; Precio ;STOCK;CATEGORIA</strong>.
                    </p>
                    <input type="file" id="csv-file-input" accept=".csv" class="w-full text-sm text-gray-500
                        file:mr-4 file:py-2 file:px-4
                        file:rounded-full file:border-0
                        file:text-sm file:font-semibold
                        file:bg-blue-50 file:text-blue-700
                        hover:file:bg-blue-100
                    "/>
                    <button id="upload-csv-btn" class="w-full bg-blue-600 text-white font-bold py-2 rounded-lg hover:bg-blue-700 transition">Cargar archivo</button>
                    <div id="upload-message" class="mt-2 text-sm text-green-600"></div>
                </div>

                <div class="overflow-x-auto">
                    <table class="min-w-full bg-white border border-gray-200 rounded-lg">
                        <thead class="bg-gray-100">
                            <tr>
                                <th class="py-2 px-4 border-b">Nombre</th>
                                <th class="py-2 px-4 border-b">Precio</th>
                                <th class="py-2 px-4 border-b">Stock</th>
                                <th class="py-2 px-4 border-b">Acciones</th>
                            </tr>
                        </thead>
                        <tbody id="products-table-body"></tbody>
                    </table>
                </div>
            `;
            const tableBody = document.getElementById("products-table-body");
            stockData.forEach((product, index) => {
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td class="py-2 px-4 border-b">${product.nombre}</td>
                    <td class="py-2 px-4 border-b">
                        <input type="text" value="${product[' Precio ']}" data-index="${index}" class="w-32 border rounded px-2 py-1 text-center price-input">
                    </td>
                    <td class="py-2 px-4 border-b">
                        <input type="number" value="${product.STOCK}" data-index="${index}" class="w-20 border rounded px-2 py-1 text-center stock-input">
                    </td>
                    <td class="py-2 px-4 border-b">
                        <button class="bg-green-500 text-white px-3 py-1 rounded-lg hover:bg-green-600 transition save-btn" data-index="${index}">Guardar</button>
                    </td>
                `;
                tableBody.appendChild(row);
            });
            
            adminContentSection.querySelectorAll('.save-btn').forEach(button => {
                button.addEventListener('click', (e) => {
                    const index = e.target.dataset.index;
                    const stockInput = adminContentSection.querySelector(`.stock-input[data-index="${index}"]`);
                    const priceInput = adminContentSection.querySelector(`.price-input[data-index="${index}"]`);
                    
                    stockData[index].STOCK = stockInput.value;
                    stockData[index][' Precio '] = priceInput.value;
                    
                    console.log(`¡Stock y precio actualizados para ${stockData[index].nombre}! Nuevo stock: ${stockData[index].STOCK}, Nuevo precio: ${stockData[index][' Precio ']}`);
                });
            });

            // Lógica de subida de archivo
            const csvFileInput = document.getElementById('csv-file-input');
            const uploadBtn = document.getElementById('upload-csv-btn');
            const uploadMessage = document.getElementById('upload-message');

            uploadBtn.addEventListener('click', () => {
                const file = csvFileInput.files[0];
                if (!file) {
                    uploadMessage.textContent = "Por favor, selecciona un archivo CSV.";
                    uploadMessage.className = "mt-2 text-sm text-red-600";
                    return;
                }

                const reader = new FileReader();
                reader.onload = (event) => {
                    try {
                        const newStockData = parseCSV(event.target.result);
                        if (newStockData.length > 0 && 'nombre' in newStockData[0] && ' Precio ' in newStockData[0] && 'STOCK' in newStockData[0] && 'CATEGORIA' in newStockData[0]) {
                            stockData = newStockData;
                            renderProductsView(); // Volver a renderizar la tabla
                            uploadMessage.textContent = "Archivo cargado y stock actualizado correctamente.";
                            uploadMessage.className = "mt-2 text-sm text-green-600";
                        } else {
                            uploadMessage.textContent = "Formato de archivo incorrecto. Asegúrate de que las columnas son: 'nombre; Precio ;STOCK;CATEGORIA'";
                            uploadMessage.className = "mt-2 text-sm text-red-600";
                        }
                    } catch (e) {
                        uploadMessage.textContent = "Error al procesar el archivo. Verifica el formato.";
                        uploadMessage.className = "mt-2 text-sm text-red-600";
                    }
                };
                reader.onerror = () => {
                    uploadMessage.textContent = "Error al leer el archivo.";
                    uploadMessage.className = "mt-2 text-sm text-red-600";
                };
                reader.readAsText(file);
            });
        }

        // Render orders: mostrará tanto los pedidos CSV como los pedidos locales (objetos)
        function renderOrdersView() {
            adminContentSection.innerHTML = `
                <h3 class="text-xl font-semibold mb-4 text-gray-700">Resumen de Pedidos</h3>
                <div class="flex justify-end mb-4 space-x-2">
                    <button id="download-orders-csv-btn" class="bg-purple-600 text-white font-bold py-2 px-4 rounded-lg hover:bg-purple-700 transition">
                        Descargar CSV
                    </button>
                    <button id="download-orders-json-btn" class="bg-indigo-600 text-white font-bold py-2 px-4 rounded-lg hover:bg-indigo-700 transition">
                        Descargar JSON
                    </button>
                    <button id="clear-local-orders-btn" class="bg-red-500 text-white font-bold py-2 px-4 rounded-lg hover:bg-red-600 transition">
                        Borrar Pedidos Locales
                    </button>
                </div>
                <div class="overflow-x-auto">
                    <table class="min-w-full bg-white border border-gray-200 rounded-lg">
                        <thead class="bg-gray-100">
                            <tr>
                                <th class="py-2 px-4 border-b">Fecha</th>
                                <th class="py-2 px-4 border-b">Nombre</th>
                                <th class="py-2 px-4 border-b">Contacto</th>
                                <th class="py-2 px-4 border-b">Producto</th>
                                <th class="py-2 px-4 border-b">Cantidad</th>
                                <th class="py-2 px-4 border-b">Precio</th>
                            </tr>
                        </thead>
                        <tbody id="orders-table-body"></tbody>
                    </table>
                </div>
            `;
            const tableBody = document.getElementById("orders-table-body");
            tableBody.innerHTML = '';

            // Primero renderizamos los pedidos CSV (objetos planos)
            ordersData.forEach(order => {
                // Si el order tiene la propiedad 'items' lo consideramos un pedido local (obj con items), si no, es CSV historic
                if (order && order.items && Array.isArray(order.items)) {
                    // pedido local: uno por cada item
                    order.items.forEach(it => {
                        const row = document.createElement('tr');
                        row.innerHTML = `
                            <td class="py-2 px-4 border-b">${order.Timestamp || order.timestamp}</td>
                            <td class="py-2 px-4 border-b">${order['Nombre y apellido'] || order.nombre || ''}</td>
                            <td class="py-2 px-4 border-b">${order.contacto || order['Contacto'] || ''}</td>
                            <td class="py-2 px-4 border-b">${it.name}</td>
                            <td class="py-2 px-4 border-b">${it.qty}</td>
                            <td class="py-2 px-4 border-b">${it.price !== undefined ? it.price : ''}</td>
                        `;
                        tableBody.appendChild(row);
                    });
                } else {
                    // pedido proveniente del CSV: iteramos sus keys a partir de la 4ª columna (como en tu versión original)
                    if (typeof order === 'object') {
                        Object.keys(order).slice(0).forEach(productName => {
                            // consideramos que las columnas de producto tienen valores numéricos o no vacíos
                            if (productName === 'Timestamp' || productName === 'Nombre y apellido' || productName === 'Telefono/Mail' || productName === '  Área / Sector  ') return;
                            if (order[productName] && order[productName].trim() !== '0' && order[productName].trim() !== '') {
                                const row = document.createElement('tr');
                                row.innerHTML = `
                                    <td class="py-2 px-4 border-b">${order.Timestamp}</td>
                                    <td class="py-2 px-4 border-b">${order['Nombre y apellido']}</td>
                                    <td class="py-2 px-4 border-b">${order['Telefono/Mail'] || ''}</td>
                                    <td class="py-2 px-4 border-b">${productName}</td>
                                    <td class="py-2 px-4 border-b">${order[productName]}</td>
                                    <td class="py-2 px-4 border-b"></td>
                                `;
                                tableBody.appendChild(row);
                            }
                        });
                    }
                }
            });

            // Botones de descarga & limpiar
            document.getElementById('download-orders-csv-btn').addEventListener('click', () => {
                const rows = [];
                // cabecera
                rows.push(['Timestamp','Nombre','Contacto','Producto','Cantidad','Precio']);
                // recorremos ordersData y también localStorage (aseguramos no duplicar)
                const processed = new Set();
                ordersData.forEach((order, idx) => {
                    if (order && order.items && Array.isArray(order.items)) {
                        order.items.forEach(it => {
                            rows.push([order.Timestamp || order.timestamp || new Date().toLocaleString(), order['Nombre y apellido'] || order.nombre || '', order.contacto || '', it.name, it.qty, it.price || '']);
                        });
                    } else if (typeof order === 'object') {
                        // CSV historic: buscar claves de producto a partir de la 4ta columna
                        Object.keys(order).slice(4).forEach(productName => {
                            if (order[productName] && order[productName].trim() !== '0' && order[productName].trim() !== '') {
                                rows.push([order.Timestamp, order['Nombre y apellido'], order['Telefono/Mail'] || '', productName, order[productName], '']);
                            }
                        });
                    }
                });

                const csvContent = "data:text/csv;charset=utf-8," + rows.map(r => r.map(c => `"${(c||'').toString().replace(/"/g,'""')}"`).join(';')).join('\n');
                const encodedUri = encodeURI(csvContent);
                const link = document.createElement("a");
                link.setAttribute("href", encodedUri);
                link.setAttribute("download", "pedidos_completos.csv");
                document.body.appendChild(link);
                link.click();
                document.body.removeChild(link);
            });

            document.getElementById('download-orders-json-btn').addEventListener('click', () => {
                const json = JSON.stringify(ordersData, null, 2);
                const blob = new Blob([json], {type: 'application/json'});
                const url = URL.createObjectURL(blob);
                const link = document.createElement('a');
                link.href = url;
                link.download = 'pedidos_completos.json';
                document.body.appendChild(link);
                link.click();
                document.body.removeChild(link);
                URL.revokeObjectURL(url);
            });

            document.getElementById('clear-local-orders-btn').addEventListener('click', () => {
                if (!confirm('¿Borrar todos los pedidos almacenados localmente? Esta acción no afecta los pedidos históricos CSV.')) return;
                localStorage.removeItem(LOCAL_ORDERS_KEY);
                // quitar del arreglo ordersData los objetos locales (identificamos por la prop 'items')
                ordersData = ordersData.filter(o => !(o && o.items && Array.isArray(o.items)));
                renderOrdersView(); // volver a renderizar
            });
        }

        // -----------------------
        // Shop / Products / Cart
        // -----------------------
        function renderUserShop() {
            shopMessage.classList.add('hidden');
            categoryList.innerHTML = '';
            const categories = [...new Set(stockData.map(p => p.CATEGORIA))];
            categories.forEach(category => {
                const categoryBtn = document.createElement('button');
                categoryBtn.textContent = category;
                categoryBtn.className = 'bg-gray-200 text-gray-700 font-bold py-2 px-4 rounded-lg hover:bg-gray-300 transition';
                categoryBtn.addEventListener('click', () => renderProducts(category));
                categoryList.appendChild(categoryBtn);
            });
            productList.classList.add('hidden');
            backToCategoriesBtn.classList.add('hidden');
            categoryList.classList.remove('hidden');
            if (dniInput.value.trim() === ADMIN_DNI) {
                backToAdminBtn.classList.remove('hidden');
            } else {
                backToAdminBtn.classList.add('hidden');
            }
        }

        function renderProducts(category) {
            shopMessage.classList.add('hidden');
            categoryList.classList.add('hidden');
            productList.classList.remove('hidden');
            productList.innerHTML = '';
            backToCategoriesBtn.classList.remove('hidden');

            const filteredProducts = stockData.filter(p => p.CATEGORIA === category);
            filteredProducts.forEach(product => {
                const productCard = document.createElement('div');
                const priceText = (product[' Precio '] || '').replace('$', '').replace(/\./g, '').replace(',', '.');
                const numericPrice = parseFloat(priceText) || 0;
                
                productCard.className = 'bg-white p-4 rounded-lg shadow-md flex flex-col items-center text-center transition hover:shadow-lg';
                productCard.innerHTML = `
                    <h3 class="font-bold text-gray-800 text-lg mb-2">${product.nombre}</h3>
                    <p class="text-sm text-gray-600 mb-2">Categoría: ${product.CATEGORIA}</p>
                    <p class="text-2xl font-bold text-green-600 mb-2">${product[' Precio ']}</p>
                    <p class="text-sm text-gray-500 mb-4">Stock: ${product.STOCK}</p>
                    <button class="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600 transition add-to-cart-btn"
                        data-name="${product.nombre}"
                        data-price="${numericPrice}"
                        data-stock="${product.STOCK}">
                        Agregar al carrito
                    </button>
                `;
                productList.appendChild(productCard);
            });
            
            productList.querySelectorAll('.add-to-cart-btn').forEach(button => {
                button.addEventListener('click', (e) => {
                    const name = e.target.dataset.name;
                    const price = parseFloat(e.target.dataset.price);
                    addToCart(name, price);
                });
            });
        }
        
        // Funcionalidad del Carrito
        function addToCart(name, price) {
            const existingItem = cart.find(item => item.name === name);
            const productInStock = stockData.find(p => p.nombre === name);

            if (!productInStock) {
                showMessage("Producto no encontrado.", 'red');
                return;
            }

            if (Number(productInStock.STOCK) <= 0) {
                showMessage("No hay stock disponible de este producto.", 'red');
                return;
            }

            if (existingItem) {
                existingItem.quantity++;
            } else {
                cart.push({ name, price, quantity: 1 });
            }
            productInStock.STOCK = Number(productInStock.STOCK) - 1; // Reduce el stock localmente
            updateCartCount();
            showMessage(`"${name}" añadido al carrito.`, 'green');
            renderProducts(productInStock.CATEGORIA); // Vuelve a renderizar la lista de productos
        }
        
        function updateCartCount() {
            const totalItems = cart.reduce((sum, item) => sum + item.quantity, 0);
            cartCount.textContent = totalItems;
        }

        function updateCartUI() {
            cartItemsList.innerHTML = '';
            let total = 0;
            if (cart.length === 0) {
                cartItemsList.innerHTML = '<li class="text-gray-500">El carrito está vacío.</li>';
            } else {
                cart.forEach(item => {
                    const listItem = document.createElement('li');
                    listItem.className = 'flex items-center justify-between p-2 bg-gray-100 rounded-lg';
                    const subtotal = item.quantity * item.price;
                    total += subtotal;
                    
                    listItem.innerHTML = `
                        <div>
                            <p class="font-semibold">${item.name}</p>
                            <p class="text-sm text-gray-600">$${(item.price * item.quantity).toFixed(2)} (${item.quantity} x $${item.price.toFixed(2)})</p>
                        </div>
                        <div class="flex items-center space-x-2">
                            <button class="bg-blue-500 text-white w-6 h-6 rounded-full flex items-center justify-center text-xs" data-name="${item.name}" data-action="decrease">-</button>
                            <span class="font-bold">${item.quantity}</span>
                            <button class="bg-blue-500 text-white w-6 h-6 rounded-full flex items-center justify-center text-xs" data-name="${item.name}" data-action="increase">+</button>
                            <button class="bg-red-500 text-white w-6 h-6 rounded-full flex items-center justify-center text-xs" data-name="${item.name}" data-action="remove">X</button>
                        </div>
                    `;
                    cartItemsList.appendChild(listItem);
                });
            }
            cartTotalPrice.textContent = `$${total.toFixed(2)}`;
        }
        
        function handleCartAction(e) {
            const button = e.target.closest('button');
            if (!button) return;
            
            const name = button.dataset.name;
            const action = button.dataset.action;
            const item = cart.find(i => i.name === name);
            const productInStock = stockData.find(p => p.nombre === name);
            
            if (!item) return;

            if (action === 'increase') {
                if (productInStock && Number(productInStock.STOCK) > 0) {
                    item.quantity++;
                    productInStock.STOCK = Number(productInStock.STOCK) - 1;
                } else {
                    showMessage("No hay más stock para este producto.", 'red');
                }
            } else if (action === 'decrease') {
                if (item.quantity > 1) {
                    item.quantity--;
                    if (productInStock) productInStock.STOCK = Number(productInStock.STOCK) + 1;
                }
            } else if (action === 'remove') {
                if (productInStock) productInStock.STOCK = Number(productInStock.STOCK) + item.quantity; // Devolver stock
                cart = cart.filter(i => i.name !== name);
            }
            updateCartCount();
            updateCartUI();
        }
        
        function showMessage(text, type = 'green') {
            shopMessage.textContent = text;
            shopMessage.classList.remove('hidden');
            shopMessage.classList.remove('bg-green-100', 'bg-red-100', 'text-green-600', 'text-red-600');
            shopMessage.classList.add(`bg-${type}-100`, `text-${type}-600`);
            setTimeout(() => {
                shopMessage.classList.add('hidden');
            }, 3000);
        }

        function goBackToCategories() {
            productList.classList.add('hidden');
            backToCategoriesBtn.classList.add('hidden');
            categoryList.classList.remove('hidden');
        }

        // Eventos
        document.getElementById('admin-users-btn').addEventListener('click', renderUsersView);
        document.getElementById('admin-products-btn').addEventListener('click', renderProductsView);
        document.getElementById('admin-orders-btn').addEventListener('click', renderOrdersView);
        document.getElementById('go-to-shop-btn').addEventListener('click', () => {
            adminPanel.classList.add('hidden');
            userShop.classList.remove('hidden');
            renderUserShop();
        });
        document.getElementById('back-to-admin-btn').addEventListener('click', () => {
            userShop.classList.add('hidden');
            adminPanel.classList.remove('hidden');
            renderAdminDashboard();
        });
        loginForm.addEventListener('submit', handleLogin);
        backToCategoriesBtn.addEventListener('click', goBackToCategories);
        
        // Eventos del carrito
        viewCartBtn.addEventListener('click', () => {
            cartModal.classList.remove('hidden');
            updateCartUI();
        });
        closeCartBtn.addEventListener('click', () => {
            cartModal.classList.add('hidden');
        });
        clearCartBtn.addEventListener('click', () => {
            // Devolver stock a los productos
            cart.forEach(item => {
                const product = stockData.find(p => p.nombre === item.name);
                if (product) {
                    product.STOCK = Number(product.STOCK) + item.quantity;
                }
            });
            cart = [];
            updateCartCount();
            updateCartUI();
            cartModal.classList.add('hidden');
        });

        // Función que guarda el pedido en localStorage y (si corresponde) en Firestore
        async function saveOrder(orderObj) {
            // Guardar localmente
            const local = loadLocalOrders();
            local.push(orderObj);
            saveLocalOrders(local);

            // Añadir también a ordersData en memoria para que se vea en la tabla sin recargar
            ordersData.push(orderObj);

            // Si hay Firebase, intentamos subir
            if (db) {
                try {
                    await addDoc(collection(db, 'orders'), {
                        ...orderObj,
                        createdAt: new Date().toISOString()
                    });
                    console.log('Pedido guardado en Firestore');
                } catch (e) {
                    console.warn('No se pudo guardar en Firestore:', e);
                }
            }
        }

        checkoutBtn.addEventListener('click', async () => {
            if (cart.length === 0) {
                showMessage("El carrito está vacío. No se puede realizar el pedido.", 'red');
                return;
            }

            // Pedimos contacto al usuario (teléfono o mail). Usamos prompt para no bloquear flujo.
            const contacto = prompt("Ingresa tu teléfono o email para contacto (opcional):", "");

            const now = new Date();
            const timestamp = now.toLocaleString();

            const nombreUsuario = userNameDisplay.textContent || (dniInput.value.trim() || 'Anon');

            // Construimos objeto pedido normalizado
            const orderObj = {
                Timestamp: timestamp,
                'Nombre y apellido': nombreUsuario,
                contacto: contacto || '',
                items: cart.map(i => ({ name: i.name, qty: i.quantity, price: i.price })),
                total: cart.reduce((s,i)=>s + i.price * i.quantity, 0),
                userId: userIdDisplay.textContent || ''
            };

            // Guardar
            await saveOrder(orderObj);

            showMessage("¡Pedido realizado con éxito! Un administrador se pondrá en contacto contigo.", 'green');

            // limpiar carrito
            cart = [];
            updateCartCount();
            updateCartUI();
            cartModal.classList.add('hidden');
        });

        cartItemsList.addEventListener('click', handleCartAction);

        // Inicialización final
        (async () => {
            // Cargar pedidos locales y fusionarlos
            const local = loadLocalOrders();
            if (local.length > 0) {
                local.forEach(o => ordersData.push(o));
            }
            await initializeFirebase();
        })();

    </script>
</body>
</html>
