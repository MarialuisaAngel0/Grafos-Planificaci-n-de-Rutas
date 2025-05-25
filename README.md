!DOCTYPE html>
<html lang="es">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Planificador de Rutas de Entrega</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        #map {
            height: 500px;
            border-radius: 0.5rem;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
        }

        .node-marker {
            background-color: #3B82F6;
            border-radius: 50%;
            width: 20px;
            height: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            color: white;
            font-weight: bold;
            font-size: 12px;
        }

        .path-marker {
            background-color: #10B981;
            border-radius: 50%;
            width: 15px;
            height: 15px;
            display: flex;
            justify-content: center;
            align-items: center;
            color: white;
            font-weight: bold;
            font-size: 10px;
        }

        .route-line {
            stroke: #3B82F6;
            stroke-width: 4;
            stroke-dasharray: 10;
            fill: none;
        }

        .package-item {
            transition: all 0.2s ease;
        }

        .package-item:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
        }
    </style>
</head>

<body class="bg-gray-50 min-h-screen">
    <div class="container mx-auto px-4 py-8">
        <header class="mb-8">
            <h1 class="text-3xl font-bold text-gray-800 flex items-center">
                <i class="fas fa-truck-fast mr-3 text-blue-500"></i>
                Planificador de Rutas de Entrega
            </h1>
            <p class="text-gray-600 mt-2">Optimiza tus entregas usando el algoritmo de Dijkstra</p>
        </header>

        <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
            <!-- Panel izquierdo - Formulario y paquetes -->
            <div class="lg:col-span-1 space-y-6">
                <div class="bg-white rounded-lg shadow p-6">
                    <h2 class="text-xl font-semibold text-gray-800 mb-4 flex items-center">
                        <i class="fas fa-plus-circle mr-2 text-blue-500"></i>
                        Agregar Paquete
                    </h2>
                    <form id="packageForm" class="space-y-4">
                        <div>
                            <label for="packageId" class="block text-sm font-medium text-gray-700">ID del
                                Paquete</label>
                            <input type="text" id="packageId"
                                class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 p-2 border">
                        </div>
                        <div>
                            <label for="destination" class="block text-sm font-medium text-gray-700">Punto de
                                Entrega</label>
                            <select id="destination"
                                class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 p-2 border">
                                <option value="">Seleccione un destino</option>
                            </select>
                        </div>
                        <div>
                            <label for="priority" class="block text-sm font-medium text-gray-700">Prioridad</label>
                            <select id="priority"
                                class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 p-2 border">
                                <option value="normal">Normal</option>
                                <option value="high">Alta</option>
                                <option value="urgent">Urgente</option>
                            </select>
                        </div>
                        <button type="submit"
                            class="w-full bg-blue-500 hover:bg-blue-600 text-white font-medium py-2 px-4 rounded-md transition duration-150 ease-in-out flex items-center justify-center">
                            <i class="fas fa-box-circle-check mr-2"></i>
                            Agregar Paquete
                        </button>
                    </form>
                </div>

                <div class="bg-white rounded-lg shadow p-6">
                    <div class="flex justify-between items-center mb-4">
                        <h2 class="text-xl font-semibold text-gray-800 flex items-center">
                            <i class="fas fa-boxes-stacked mr-2 text-blue-500"></i>
                            Paquetes para Entrega
                        </h2>
                        <span id="packageCount"
                            class="bg-blue-100 text-blue-800 text-xs font-medium px-2.5 py-0.5 rounded-full">0</span>
                    </div>
                    <div id="packagesList" class="space-y-3 max-h-96 overflow-y-auto pr-2">
                        <p class="text-gray-500 text-center py-4">No hay paquetes agregados</p>
                    </div>
                </div>
            </div>

            <!-- Panel derecho - Mapa y resultados -->
            <div class="lg:col-span-2 space-y-6">
                <div class="bg-white rounded-lg shadow p-6">
                    <div class="flex justify-between items-center mb-4">
                        <h2 class="text-xl font-semibold text-gray-800 flex items-center">
                            <i class="fas fa-map-location-dot mr-2 text-blue-500"></i>
                            Mapa de Rutas
                        </h2>
                        <button id="planRouteBtn"
                            class="bg-green-500 hover:bg-green-600 text-white font-medium py-2 px-4 rounded-md transition duration-150 ease-in-out flex items-center disabled:opacity-50"
                            disabled>
                            <i class="fas fa-route mr-2"></i>
                            Planificar Ruta
                        </button>
                    </div>
                    <div id="map"></div>
                </div>

                <div class="bg-white rounded-lg shadow p-6">
                    <h2 class="text-xl font-semibold text-gray-800 mb-4 flex items-center">
                        <i class="fas fa-info-circle mr-2 text-blue-500"></i>
                        Detalles de la Ruta
                    </h2>
                    <div id="routeDetails" class="space-y-4">
                        <div class="flex items-center justify-center py-8">
                            <p class="text-gray-500">No se ha calculado ninguna ruta</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // Grafo de ejemplo (nodos y conexiones)
        const graph = {
            'Almacén': {
                'Xetulul': 5,
                'Xocomil': 2
            },
            'Xetulul': {
                'Museo de Arqueología': 4,
                'Terminal de buses': 2
            },
            'Xocomil': {
                'Xetulul': 8,
                'Terminal de buses': 7
            },
            'Museo de Arqueología': {
                'Terminal de buses': 6,
                'Otra ubicación cercana': 3
            },
            'Terminal de buses': {
                'Otra ubicación cercana': 1
            },
            'Otra ubicación cercana': {}
        };

        // Coordenadas aproximadas para el mapa (lat, lng)
        // const nodeCoordinates = {
        //     'Almacén': [51.505, -0.09],
        //     'Xetulul': [51.51, -0.1],
        //     'Xocomil': [51.515, -0.09],
        //     'Museo de Arqueología': [51.52, -0.12],
        //     'D': [51.525, -0.1],
        //     'E': [51.53, -0.11]
        // };
        const nodeCoordinates = {
            'Almacén': [14.5339, -91.6825], // Parque Central de Retalhuleu
            'Xetulul': [14.5431, -91.6765],       // Xetulul
            'Xocomil': [14.5400, -91.6770],       // Xocomil
            'Museo de Arqueología': [14.5317, -91.6848],       // Museo de Arqueología
            'Terminal de buses': [14.5346, -91.6791],       // Terminal de buses
            'Otra ubicación cercana': [14.5365, -91.6850]        // Otra ubicación cercana
        };


        // Almacén de paquetes
        let packages = [];

        // Inicializar mapa
        const map = L.map('map').setView([14.5366, -91.6770], 13);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
        }).addTo(map);

        // Marcadores y líneas del grafo
        const markers = {};
        const lines = [];

        // Dibujar el grafo en el mapa
        function drawGraph() {
            // Limpiar marcadores y líneas existentes
            Object.values(markers).forEach(marker => map.removeLayer(marker));
            lines.forEach(line => map.removeLayer(line));

            // Agregar nodos al select de destinos
            const destinationSelect = document.getElementById('destination');
            destinationSelect.innerHTML = '<option value="">Seleccione un destino</option>';

            Object.keys(graph).forEach(node => {
                if (node !== 'Almacén') {
                    const option = document.createElement('option');
                    option.value = node;
                    option.textContent = node;
                    destinationSelect.appendChild(option);
                }

                // Crear marcador para el nodo
                const marker = L.marker(nodeCoordinates[node], {
                    icon: L.divIcon({
                        className: node === 'Almacén' ? 'node-marker warehouse' : 'node-marker',
                        html: node === 'Almacén' ? '<i class="fas fa-warehouse"></i>' : node,
                        iconSize: [20, 20]
                    })
                }).addTo(map);

                marker.bindPopup(<b>${node}</b>);
                markers[node] = marker;
            });

            // Dibujar conexiones (aristas)
            Object.keys(graph).forEach(fromNode => {
                Object.keys(graph[fromNode]).forEach(toNode => {
                    const weight = graph[fromNode][toNode];
                    const line = L.polyline(
                        [nodeCoordinates[fromNode], nodeCoordinates[toNode]],
                        { color: '#6B7280', weight: 2, dashArray: '5, 5' }
                    ).addTo(map);

                    // Agregar etiqueta de peso
                    const center = [
                        (nodeCoordinates[fromNode][0] + nodeCoordinates[toNode][0]) / 2,
                        (nodeCoordinates[fromNode][1] + nodeCoordinates[toNode][1]) / 2
                    ];

                    const label = L.marker(center, {
                        icon: L.divIcon({
                            className: 'path-marker',
                            html: weight,
                            iconSize: [15, 15]
                        }),
                        zIndexOffset: 1000
                    }).addTo(map);

                    lines.push(line);
                    lines.push(label);
                });
            });
        }

        // Implementación del algoritmo de Dijkstra
        function dijkstra(graph, start) {
            const distances = {};
            const previous = {};
            const nodes = new Set();
            const visited = new Set();

            // Inicialización
            for (const node in graph) {
                distances[node] = node === start ? 0 : Infinity;
                previous[node] = null;
                nodes.add(node);
            }

            while (nodes.size) {
                // Encontrar el nodo con la distancia mínima
                let minNode = null;
                for (const node of nodes) {
                    if (minNode === null || distances[node] < distances[minNode]) {
                        minNode = node;
                    }
                }

                if (minNode === null || distances[minNode] === Infinity) break;

                // Visitar el nodo
                nodes.delete(minNode);
                visited.add(minNode);

                // Actualizar distancias a los vecinos
                for (const neighbor in graph[minNode]) {
                    const distance = distances[minNode] + graph[minNode][neighbor];
                    if (distance < distances[neighbor]) {
                        distances[neighbor] = distance;
                        previous[neighbor] = minNode;
                    }
                }
            }

            return { distances, previous };
        }

        // Obtener la ruta más corta desde el inicio hasta el destino
        function getShortestPath(previous, destination) {
            const path = [destination];
            let current = destination;

            while (previous[current] !== null) {
                current = previous[current];
                path.unshift(current);
            }

            return path;
        }

        // Calcular la ruta óptima para todos los paquetes
        function calculateOptimalRoute() {
            if (packages.length === 0) return;

            // Agrupar paquetes por destino
            const destinations = {};
            packages.forEach(pkg => {
                if (!destinations[pkg.destination]) {
                    destinations[pkg.destination] = [];
                }
                destinations[pkg.destination].push(pkg);
            });

            // Calcular rutas desde el almacén a cada destino
            const { distances, previous } = dijkstra(graph, 'Almacén');

            // Ordenar destinos por distancia
            const sortedDestinations = Object.keys(destinations)
                .filter(dest => distances[dest] !== Infinity)
                .sort((a, b) => distances[a] - distances[b]);

            // Mostrar resultados
            const routeDetails = document.getElementById('routeDetails');
            routeDetails.innerHTML = '';

            if (sortedDestinations.length === 0) {
                routeDetails.innerHTML = `
                    <div class="bg-red-100 border-l-4 border-red-500 text-red-700 p-4">
                        <p>No se puede llegar a ninguno de los destinos desde el almacén.</p>
                    </div>
                `;
                return;
            }

            // Mostrar ruta para cada destino
            sortedDestinations.forEach((dest, index) => {
                const path = getShortestPath(previous, dest);
                const distance = distances[dest];
                const packagesForDest = destinations[dest];

                const destCard = document.createElement('div');
                destCard.className = 'bg-gray-50 rounded-lg p-4 border border-gray-200';
                destCard.innerHTML = `
                    <div class="flex justify-between items-start mb-2">
                        <h3 class="font-medium text-gray-800">
                            <span class="bg-blue-500 text-white rounded-full w-6 h-6 inline-flex items-center justify-center mr-2">${index + 1}</span>
                            ${dest}
                        </h3>
                        <span class="bg-green-100 text-green-800 text-xs font-medium px-2.5 py-0.5 rounded-full">
                            ${distance} km
                        </span>
                    </div>
                    <div class="mb-3">
                        <p class="text-sm text-gray-600">Ruta: ${path.join(' → ')}</p>
                    </div>
                    <div>
                        <p class="text-sm font-medium text-gray-700 mb-1">Paquetes:</p>
                        <ul class="space-y-1">
                            ${packagesForDest.map(pkg => `
                                <li class="text-sm text-gray-600 flex items-center">
                                    <i class="fas fa-box mr-2 text-blue-500"></i>
                                    ${pkg.id} (${pkg.priority === 'high' ? 'Alta prioridad' : pkg.priority === 'urgent' ? 'Urgente' : 'Normal'})
                                </li>
                            `).join('')}
                        </ul>
                    </div>
                `;

                routeDetails.appendChild(destCard);
            });

            // Dibujar la ruta en el mapa (primera ruta como ejemplo)
            const firstPath = getShortestPath(previous, sortedDestinations[0]);
            drawRouteOnMap(firstPath);
        }

        // Dibujar una ruta específica en el mapa
        function drawRouteOnMap(path) {
            // Limpiar rutas anteriores
            const oldRouteLines = document.querySelectorAll('.route-line');
            oldRouteLines.forEach(line => line.remove());

            // Dibujar nueva ruta
            for (let i = 0; i < path.length - 1; i++) {
                const fromNode = path[i];
                const toNode = path[i + 1];

                // Resaltar conexión en el mapa
                const line = L.polyline(
                    [nodeCoordinates[fromNode], nodeCoordinates[toNode]],
                    { color: '#3B82F6', weight: 4, className: 'route-line' }
                ).addTo(map);

                // Resaltar nodos de la ruta
                markers[fromNode].setIcon(L.divIcon({
                    className: fromNode === 'Almacén' ? 'node-marker warehouse' : 'node-marker',
                    html: fromNode === 'Almacén' ? '<i class="fas fa-warehouse"></i>' : fromNode,
                    iconSize: [25, 25]
                }));

                markers[toNode].setIcon(L.divIcon({
                    className: 'node-marker',
                    html: toNode,
                    iconSize: [25, 25]
                }));
            }

            // Ajustar vista del mapa para mostrar toda la ruta
            const bounds = path.map(node => nodeCoordinates[node]);
            map.fitBounds(bounds, { padding: [50, 50] });
        }

        // Actualizar lista de paquetes en la UI
        function updatePackagesList() {
            const packagesList = document.getElementById('packagesList');
            const packageCount = document.getElementById('packageCount');
            const planRouteBtn = document.getElementById('planRouteBtn');

            if (packages.length === 0) {
                packagesList.innerHTML = '<p class="text-gray-500 text-center py-4">No hay paquetes agregados</p>';
                packageCount.textContent = '0';
                planRouteBtn.disabled = true;
                return;
            }

            packageCount.textContent = packages.length;
            planRouteBtn.disabled = false;

            packagesList.innerHTML = packages.map((pkg, index) => `
                <div class="package-item bg-gray-50 rounded-lg p-4 border border-gray-200">
                    <div class="flex justify-between items-start">
                        <div>
                            <h3 class="font-medium text-gray-800">${pkg.id}</h3>
                            <p class="text-sm text-gray-600">Destino: ${pkg.destination}</p>
                        </div>
                        <span class="text-xs font-medium px-2.5 py-0.5 rounded-full ${pkg.priority === 'high' ? 'bg-yellow-100 text-yellow-800' :
                    pkg.priority === 'urgent' ? 'bg-red-100 text-red-800' : 'bg-blue-100 text-blue-800'
                }">
                            ${pkg.priority === 'high' ? 'Alta' : pkg.priority === 'urgent' ? 'Urgente' : 'Normal'}
                        </span>
                    </div>
                    <div class="mt-3 flex justify-end">
                        <button onclick="removePackage(${index})" class="text-red-500 hover:text-red-700 text-sm flex items-center">
                            <i class="fas fa-trash-can mr-1"></i>
                            Eliminar
                        </button>
                    </div>
                </div>
            `).join('');
        }

        // Eliminar un paquete
        function removePackage(index) {
            packages.splice(index, 1);
            updatePackagesList();
        }

        // Manejar envío del formulario
        document.getElementById('packageForm').addEventListener('submit', function (e) {
            e.preventDefault();

            const packageId = document.getElementById('packageId').value.trim();
            const destination = document.getElementById('destination').value;
            const priority = document.getElementById('priority').value;

            if (!packageId || !destination) {
                alert('Por favor complete todos los campos');
                return;
            }

            // Agregar nuevo paquete
            packages.push({
                id: packageId,
                destination,
                priority
            });

            // Limpiar formulario
            document.getElementById('packageId').value = '';
            document.getElementById('destination').value = '';

            // Actualizar UI
            updatePackagesList();
        });

        // Manejar clic en el botón de planificar ruta
        document.getElementById('planRouteBtn').addEventListener('click', calculateOptimalRoute);

        // Inicializar la aplicación
        document.addEventListener('DOMContentLoaded', function () {
            drawGraph();
            updatePackagesList();
        });
    </script>
</body>

</html>
