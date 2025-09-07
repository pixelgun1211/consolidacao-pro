<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard de Logística</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #111827; color: #f3f4f6; }
        .loading-spinner { border: 4px solid rgba(255, 255, 255, 0.2); border-left-color: #4f46e5; border-radius: 50%; width: 24px; height: 24px; animation: spin 1s linear infinite; }
        @keyframes spin { to { transform: rotate(360deg); } }
        .custom-modal { display: none; position: fixed; z-index: 1000; left: 0; top: 0; width: 100%; height: 100%; overflow: auto; background-color: rgba(0,0,0,0.7); justify-content: center; align-items: center; }
        .custom-modal-content { background-color: #1f2937; padding: 25px; border-radius: 8px; box-shadow: 0 5px 15px rgba(0,0,0,0.3); max-width: 400px; text-align: center; border: 1px solid #374151; }
        .tooltip { position: absolute; text-align: center; padding: 8px; font: 12px sans-serif; background: rgba(0, 0, 0, 0.8); color: white; border-radius: 4px; pointer-events: none; opacity: 0; transition: opacity 0.2s; }
        .axis text { fill: #9ca3af; }
        .axis path, .axis line { stroke: #4b5563; }
    </style>
</head>
<body class="p-8">
    <div id="main-loader" class="flex items-center justify-center h-screen">
        <div class="text-center">
            <div class="loading-spinner mx-auto"></div>
            <p class="mt-2 text-gray-400">A carregar dados...</p>
        </div>
    </div>
    <div id="main-content" class="container mx-auto hidden">
        <header class="mb-8 flex flex-col md:flex-row md:items-end md:justify-between gap-4">
            <div>
                <h1 class="text-4xl font-bold text-white">Dashboard de Operações</h1>
                <p class="text-gray-400">Visão geral do fluxo de pacotes do dia.</p>
            </div>
            <div class="flex items-center gap-2">
                <label for="historyDate" class="text-gray-300 font-medium">Selecionar Data:</label>
                <input type="date" id="historyDate" class="bg-gray-700 border border-gray-600 text-white rounded-md p-2" aria-label="Selecionar data do histórico">
            </div>
        </header>

        <!-- Painel de KPIs -->
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
            <div class="bg-gray-800 p-6 rounded-lg shadow-lg border border-gray-700">
                <h4 class="text-gray-400 text-sm font-medium">Pacotes Recebidos</h4>
                <p id="kpi-total-packages" class="text-3xl font-bold text-white mt-2">0</p>
            </div>
            <div class="bg-gray-800 p-6 rounded-lg shadow-lg border border-gray-700">
                <h4 class="text-gray-400 text-sm font-medium">Veículos em Fila</h4>
                <p id="kpi-vehicles-waiting" class="text-3xl font-bold text-white mt-2">0</p>
            </div>
            <div class="bg-gray-800 p-6 rounded-lg shadow-lg border border-gray-700">
                <h4 class="text-gray-400 text-sm font-medium">Veículos Descarregados</h4>
                <p id="kpi-vehicles-serviced" class="text-3xl font-bold text-white mt-2">0</p>
            </div>
            <div class="bg-gray-800 p-6 rounded-lg shadow-lg border border-gray-700">
                <h4 class="text-gray-400 text-sm font-medium">Tempo Médio de Espera</h4>
                <p id="kpi-avg-wait-time" class="text-3xl font-bold text-white mt-2">-- min</p>
            </div>
        </div>

        <!-- Formulário e Filas -->
        <div class="grid grid-cols-1 lg:grid-cols-3 gap-8 mb-8">
            <div class="lg:col-span-1 p-6 bg-gray-800 rounded-lg shadow-lg border border-gray-700">
                <h2 class="text-2xl font-semibold text-white mb-4">Registrar Chegada</h2>
                <form id="entryForm" class="space-y-4" autocomplete="off">
                    <div>
                        <label for="driverName" class="block text-sm font-medium text-gray-400">Nome do Motorista</label>
                        <input type="text" id="driverName" placeholder="Nome completo" required class="mt-1 w-full p-3 border border-gray-600 bg-gray-700 text-white rounded-md shadow-sm">
                    </div>
                    <div>
                        <label for="company" class="block text-sm font-medium text-gray-400">Transportadora</label>
                        <select id="company" required class="mt-1 w-full p-3 border border-gray-600 bg-gray-700 text-white rounded-md shadow-sm">
                            <option value="">Selecione...</option>
                            <option value="Imediato">Imediato</option>
                            <option value="Pralog">Pralog</option>
                            <option value="On Time">On Time</option>
                            <option value="Hawk">Hawk</option>
                        </select>
                    </div>
                    <div>
                        <label for="packageCount" class="block text-sm font-medium text-gray-400">Quantidade de Pacotes</label>
                        <input type="number" id="packageCount" placeholder="Ex: 150" required min="1" class="mt-1 w-full p-3 border border-gray-600 bg-gray-700 text-white rounded-md shadow-sm">
                    </div>
                    <button type="submit" aria-label="Adicionar motorista à fila" class="w-full bg-indigo-600 text-white p-3 rounded-md hover:bg-indigo-500 transition shadow-md flex items-center justify-center gap-2">
                        <span id="entryButtonText">Adicionar à Fila</span>
                        <div id="entrySpinner" class="hidden loading-spinner"></div>
                    </button>
                </form>
            </div>
            <div class="lg:col-span-2 p-6 bg-gray-800 rounded-lg shadow-lg border border-gray-700">
                <h2 class="text-2xl font-semibold text-white mb-4">Controle de Filas</h2>
                <div id="queuesContainer" class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <!-- Colunas das filas serão inseridas aqui -->
                </div>
            </div>
        </div>

        <!-- Dashboard de Gráficos -->
        <div class="space-y-6">
            <div class="p-6 bg-gray-800 rounded-lg shadow-lg border border-gray-700">
                <h3 class="text-xl font-semibold text-white mb-4">Volume Planejado</h3>
                <div class="flex items-center gap-4">
                    <input type="number" id="plannedPackages" placeholder="Ex: 5000" min="0" value="0" class="w-full p-3 border border-gray-600 bg-gray-700 text-white rounded-md shadow-sm flex-grow">
                    <button id="savePlannedPackagesBtn" class="bg-indigo-600 text-white p-3 rounded-md hover:bg-indigo-500 transition shadow-md whitespace-nowrap">
                        Definir Meta
                    </button>
                </div>
            </div>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                <div class="bg-gray-800 p-6 rounded-lg shadow-lg border border-gray-700 text-center">
                    <h3 class="text-xl font-semibold text-white mb-2">Progresso da Meta Diária</h3>
                    <div id="donutChartContainer" class="flex justify-center items-center h-48"></div>
                </div>
                <div class="bg-gray-800 p-6 rounded-lg shadow-lg border border-gray-700 text-center">
                    <h3 class="text-xl font-semibold text-white mb-2">Pacotes por Transportadora</h3>
                    <div id="companyDonutChartContainer" class="flex justify-center items-center h-48"></div>
                </div>
            </div>
            <div class="bg-gray-800 p-6 rounded-lg shadow-lg border border-gray-700">
                 <h3 class="text-xl font-semibold text-white mb-4 text-center">Entrada de Pacotes por Hora</h3>
                 <div id="barChartContainer" class="overflow-x-auto">
                     <div id="barChartPlaceholder" class="text-center text-gray-400 py-16">Aguardando dados...</div>
                 </div>
            </div>
        </div>
        
        <!-- Gerenciamento de Dados -->
        <div class="mt-8 p-6 bg-gray-800 rounded-lg shadow-lg border border-gray-700">
            <h2 class="text-2xl font-semibold text-white mb-4">Gerenciamento de Dados</h2>
            <div class="flex flex-col sm:flex-row justify-center gap-4">
                <button id="exportCsvBtn" aria-label="Exportar dados para Excel" class="bg-green-600 text-white p-3 rounded-md hover:bg-green-700 transition shadow-md flex items-center justify-center gap-2">
                    <span id="exportButtonText">Baixar Planilha (Excel)</span>
                    <div id="exportSpinner" class="hidden loading-spinner"></div>
                </button>
                <button id="resetDayBtn" aria-label="Resetar dados do dia" class="bg-red-600 text-white p-3 rounded-md hover:bg-red-700 transition shadow-md flex items-center justify-center gap-2">
                    <span id="resetButtonText">Resetar Dados do Dia</span>
                    <div id="resetSpinner" class="hidden loading-spinner"></div>
                </button>
            </div>
        </div>
    </div>

    <!-- Modal para Notificações -->
    <div id="customModal" class="custom-modal">
        <div class="custom-modal-content">
            <h3 id="modalTitle" class="text-xl font-bold text-white"></h3>
            <p id="modalMessage" class="my-4 text-gray-300"></p>
            <p id="modalQueueNumber" class="text-6xl font-bold text-indigo-500 my-4"></p>
            <div id="modal-buttons" class="flex justify-center gap-4">
                <button id="modalConfirmBtn" class="bg-indigo-600 text-white py-2 px-6 rounded-md hover:bg-indigo-500">OK</button>
                <button id="modalCancelBtn" class="bg-gray-500 text-white py-2 px-6 rounded-md hover:bg-gray-600 hidden">Cancelar</button>
            </div>
        </div>
    </div>

    <!-- D3.js -->
    <script src="https://d3js.org/d3.v7.min.js"></script>
    <!-- Firebase + Dashboard -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/12.2.1/firebase-app.js";
        import { getAnalytics } from "https://www.gstatic.com/firebasejs/12.2.1/firebase-analytics.js";
        import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/12.2.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, query, where, onSnapshot, doc, updateDoc, Timestamp, getDocs, writeBatch, runTransaction } from "https://www.gstatic.com/firebasejs/12.2.1/firebase-firestore.js";

        // Configuração Firebase
        const firebaseConfig = {
            apiKey: "AIzaSyAYQs0PBdu4xXG3QrfkHlCMSx3ffSQPAHg",
            authDomain: "consolidacao-86ffc.firebaseapp.com",
            projectId: "consolidacao-86ffc",
            storageBucket: "consolidacao-86ffc.appspot.com",
            messagingSenderId: "579827881426",
            appId: "1:579827881426:web:f802c332f663bc8ed5640b",
            measurementId: "G-N2S0H05R2X"
        };

        // Inicialização
        const app = initializeApp(firebaseConfig);
        getAnalytics(app);
        const auth = getAuth(app);
        const db = getFirestore(app);

        // Estado
        const COMPANIES = ["Imediato", "Pralog", "On Time", "Hawk"];
        let plannedPackages = 0;
        let allEntries = [];
        let selectedDate = null;
        let unsubscribe = null;

        // UI
        const loader = document.getElementById('main-loader');
        const content = document.getElementById('main-content');
        const dateInput = document.getElementById('historyDate');
        const kpiTotal = document.getElementById('kpi-total-packages');
        const kpiWaiting = document.getElementById('kpi-vehicles-waiting');
        const kpiServiced = document.getElementById('kpi-vehicles-serviced');
        const kpiAvgWait = document.getElementById('kpi-avg-wait-time');
        const queuesContainer = document.getElementById('queuesContainer');
        const entryForm = document.getElementById('entryForm');
        const driverNameInput = document.getElementById('driverName');
        const companySelect = document.getElementById('company');
        const packageCountInput = document.getElementById('packageCount');
        const entryButtonText = document.getElementById('entryButtonText');
        const entrySpinner = document.getElementById('entrySpinner');
        const plannedPackagesInput = document.getElementById('plannedPackages');
        const savePlannedPackagesBtn = document.getElementById('savePlannedPackagesBtn');
        const exportCsvBtn = document.getElementById('exportCsvBtn');
        const exportButtonText = document.getElementById('exportButtonText');
        const exportSpinner = document.getElementById('exportSpinner');
        const resetDayBtn = document.getElementById('resetDayBtn');
        const resetButtonText = document.getElementById('resetButtonText');
        const resetSpinner = document.getElementById('resetSpinner');
        const modal = document.getElementById('customModal');
        const modalTitle = document.getElementById('modalTitle');
        const modalMessage = document.getElementById('modalMessage');
        const modalQueueNumber = document.getElementById('modalQueueNumber');
        const modalConfirmBtn = document.getElementById('modalConfirmBtn');
        const modalCancelBtn = document.getElementById('modalCancelBtn');
        const donutChartContainer = d3.select("#donutChartContainer");
        const companyDonutChartContainer = d3.select("#companyDonutChartContainer");
        const barChartContainer = d3.select("#barChartContainer");
        const tooltip = d3.select("body").append("div").attr("class", "tooltip");

        // Utilitário para normalizar data para meia-noite local
        function getDayRange(dateStr) {
            const d = dateStr ? new Date(dateStr) : new Date();
            d.setHours(0,0,0,0);
            const start = Timestamp.fromDate(d);
            const end = Timestamp.fromDate(new Date(d.getTime() + 24*60*60*1000));
            return { start, end };
        }

        // Modal
        function showModal(title, message, options = {}) {
            return new Promise((resolve) => {
                modalTitle.textContent = title;
                modalMessage.textContent = message;
                if (options.queueNumber) {
                    modalQueueNumber.textContent = `#${options.queueNumber}`;
                    modalQueueNumber.classList.remove('hidden');
                } else {
                    modalQueueNumber.classList.add('hidden');
                }
                modalCancelBtn.classList.toggle('hidden', !options.isConfirm);
                modalConfirmBtn.textContent = options.isConfirm ? 'Confirmar' : 'OK';
                modal.style.display = 'flex';
                modalConfirmBtn.onclick = () => { modal.style.display = 'none'; resolve(true); };
                modalCancelBtn.onclick = () => { modal.style.display = 'none'; resolve(false); };
            });
        }

        // KPIs
        function updateKPIs(entries) {
            let total = 0, waiting = 0, serviced = 0, totalWait = 0;
            entries.forEach(e => {
                total += e.packageCount;
                if (e.status === 'waiting') waiting++;
                if (e.status === 'serviced') {
                    serviced++;
                    if (e.completedAt && e.timestamp) {
                        totalWait += e.completedAt.toDate() - e.timestamp.toDate();
                    }
                }
            });
            kpiTotal.textContent = total.toLocaleString('pt-BR');
            kpiWaiting.textContent = waiting;
            kpiServiced.textContent = serviced;
            if (serviced > 0) {
                kpiAvgWait.textContent = Math.round((totalWait/serviced)/60000) + " min";
            } else {
                kpiAvgWait.textContent = "-- min";
            }
        }

        // Filas
        function renderQueues(entries) {
            queuesContainer.innerHTML = '';
            COMPANIES.forEach(company => {
                const queueEl = document.createElement('div');
                queueEl.className = 'p-6 bg-gray-900 rounded-lg shadow-lg border border-gray-700';
                queueEl.innerHTML = `
                    <h3 class="text-xl font-semibold text-white mb-4">${company}</h3>
                    <ul id="queue-${company.replace(/\s/g, '')}" class="space-y-3 h-96 overflow-y-auto pr-2">
                        <li class="text-gray-400 italic">Nenhum motorista na fila.</li>
                    </ul>
                `;
                queuesContainer.appendChild(queueEl);
            });
            entries.sort((a, b) => {
                if (a.status === 'waiting' && b.status === 'serviced') return -1;
                if (a.status === 'serviced' && b.status === 'waiting') return 1;
                return a.timestamp.toDate() - b.timestamp.toDate();
            });
            entries.forEach(entry => {
                const listEl = document.getElementById(`queue-${entry.company.replace(/\s/g, '')}`);
                if (!listEl) return;
                if (listEl.querySelector('.italic')) listEl.innerHTML = '';
                const li = document.createElement('li');
                const isServiced = entry.status === 'serviced';
                li.className = `bg-gray-700 p-3 rounded-md shadow-sm border border-gray-600 flex justify-between items-center ${isServiced ? 'opacity-50' : ''}`;
                let buttonHtml = `<button class="bg-green-600 text-white px-3 py-1 rounded-md hover:bg-green-500 text-sm transition">Atendido</button>`;
                if (isServiced) {
                    buttonHtml = `<span class="text-sm font-bold text-red-500">Descarregado</span>`;
                }
                li.innerHTML = `
                    <div>
                        <div class="font-semibold text-white">#${entry.queueNumber} - ${entry.driverName}</div>
                        <div class="text-sm text-gray-300">${entry.packageCount.toLocaleString('pt-BR')} pacotes</div>
                    </div>
                    ${buttonHtml}
                `;
                if (!isServiced) {
                    li.querySelector('button').addEventListener('click', async () => {
                        await updateDoc(doc(db, "cd_shared_queue_v1", entry.id), { status: 'serviced', completedAt: Timestamp.now() });
                    });
                }
                listEl.appendChild(li);
            });
        }

        // Gráficos
        function renderCharts(entries) {
            // Meta diária
            donutChartContainer.html('');
            const totalReceived = d3.sum(entries, d => d.packageCount);
            const planned = plannedPackages;
            const width = 160, height = 160;
            const margin = 20; // Define margin for donut chart
            const radius = Math.min(width, height) / 2 - margin;
            const svg = donutChartContainer.append("svg").attr("width", width).attr("height", height).append("g").attr("transform", `translate(${width / 2}, ${height / 2})`);
            if (planned <= 0) {
                svg.append("text").attr("text-anchor", "middle").attr("class", "text-gray-400 text-xs").text("Defina uma meta.");
            } else {
                const remaining = Math.max(0, planned - totalReceived);
                const data = { Recebidos: totalReceived, Restante: remaining };
                const color = d3.scaleOrdinal().domain(Object.keys(data)).range(totalReceived >= planned ? ["#22d3ee", "#374151"] : ["#22c55e", "#374151"]);
                const pie = d3.pie().value(d => d[1]);
                const data_ready = pie(Object.entries(data));
                svg.selectAll('path').data(data_ready).join('path').attr('d', d3.arc().innerRadius(50).outerRadius(radius)).attr('fill', d => color(d.data[0])).attr("stroke", "#1f2937").style("stroke-width", "4px")
                    .on("mouseover", (event, d) => tooltip.style("opacity", 1).html(`${d.data[0]}: ${d.data[1].toLocaleString('pt-BR')} pacotes`))
                    .on("mousemove", (event) => tooltip.style("left", `${event.pageX + 15}px`).style("top", `${event.pageY - 28}px`))
                    .on("mouseout", () => tooltip.style("opacity", 0));
                const percent = (totalReceived / planned * 100).toFixed(1);
                svg.append("text").attr("text-anchor", "middle").attr("dy", "0.35em").attr("class", "text-lg font-bold text-white").text(`${percent}%`);
            }
            // Por transportadora
            companyDonutChartContainer.html('');
            const companyData = d3.rollup(entries, v => d3.sum(v, d => d.packageCount), d => d.company);
            if (companyData.size === 0) {
                companyDonutChartContainer.append('div').attr('class', 'flex items-center justify-center h-full text-gray-400 text-sm').text('Nenhum pacote registado.');
            } else {
                const svg2 = companyDonutChartContainer.append("svg").attr("width", width).attr("height", height).append("g").attr("transform", `translate(${width / 2}, ${height / 2})`);
                const color2 = d3.scaleOrdinal().domain(Array.from(companyData.keys())).range(d3.schemeTableau10);
                const pie2 = d3.pie().value(d => d[1]).sort(null);
                const data_ready2 = pie2(Array.from(companyData.entries()));
                svg2.selectAll('path').data(data_ready2).join('path').attr('d', d3.arc().innerRadius(50).outerRadius(radius)).attr('fill', d => color2(d.data[0])).attr("stroke", "#1f2937").style("stroke-width", "4px")
                    .on("mouseover", (event, d) => tooltip.style("opacity", 1).html(`${d.data[0]}: ${d.data[1].toLocaleString('pt-BR')} pacotes`))
                    .on("mousemove", (event) => tooltip.style("left", `${event.pageX + 15}px`).style("top", `${event.pageY - 28}px`))
                    .on("mouseout", () => tooltip.style("opacity", 0));
            }
            // Por hora
            barChartContainer.html('');
            d3.select("#barChartPlaceholder").remove();
            const hourlyData = d3.rollup(entries, v => d3.sum(v, d => d.packageCount), d => d.timestamp.toDate().getHours());
            const fullDayData = Array.from({length: 24}, (_, i) => ({ hour: i, load: hourlyData.get(i) || 0 }));
            const marginBar = {top: 20, right: 20, bottom: 40, left: 60};
            const widthBar = Math.max(barChartContainer.node().getBoundingClientRect().width, 500) - marginBar.left - marginBar.right;
            const heightBar = 300 - marginBar.top - marginBar.bottom;
            const svgBar = barChartContainer.append("svg").attr("width", widthBar + marginBar.left + marginBar.right).attr("height", heightBar + marginBar.top + marginBar.bottom).append("g").attr("transform", `translate(${marginBar.left},${marginBar.top})`);
            const x = d3.scaleBand().range([0, widthBar]).domain(fullDayData.map(d => `${d.hour.toString().padStart(2,'0')}:00`)).padding(0.2);
            const y = d3.scaleLinear().domain([0, d3.max(fullDayData, d => d.load) || 100]).range([heightBar, 0]);
            svgBar.append("g").attr("class", "axis").attr("transform", `translate(0,${heightBar})`).call(d3.axisBottom(x)).selectAll("text").style("text-anchor", "end").attr("transform", "rotate(-45)");
            svgBar.append("g").attr("class", "axis").call(d3.axisLeft(y).ticks(5));
            svgBar.selectAll("rect").data(fullDayData).join("rect").attr("x", d => x(`${d.hour.toString().padStart(2,'0')}:00`)).attr("y", d => y(d.load)).attr("width", x.bandwidth()).attr("height", d => heightBar - y(d.load)).attr("fill", "#4f46e5")
                .on("mouseover", (event, d) => tooltip.style("opacity", 1).html(`${d.hour.toString().padStart(2,'0')}:00 - ${d.load.toLocaleString('pt-BR')} pacotes`))
                .on("mousemove", (event) => tooltip.style("left", `${event.pageX + 15}px`).style("top", `${event.pageY - 28}px`))
                .on("mouseout", () => tooltip.style("opacity", 0));
        }

        // Listener Firestore
        function listenForDay(dateStr) {
            if (unsubscribe) unsubscribe();
            const { start, end } = getDayRange(dateStr);
            const q = query(
                collection(db, "cd_shared_queue_v1"),
                where("timestamp", ">=", start),
                where("timestamp", "<", end)
            );
            unsubscribe = onSnapshot(q, snap => {
                allEntries = [];
                snap.forEach(docSnap => {
                    const data = docSnap.data();
                    data.id = docSnap.id;
                    data.packageCount = Number(data.packageCount) || 0;
                    allEntries.push(data);
                });
                updateKPIs(allEntries);
                renderQueues(allEntries);
                renderCharts(allEntries);
            });
        }

        // Formulário de chegada
        entryForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            const driver = driverNameInput.value.trim();
            const company = companySelect.value;
            const pkgCount = parseInt(packageCountInput.value, 10);
            if (!driver || !company || !pkgCount || pkgCount < 1) {
                showModal("Atenção", "Preencha todos os campos corretamente.");
                return;
            }
            entryButtonText.classList.add('hidden');
            entrySpinner.classList.remove('hidden');
            try {
                // Garante unicidade da senha por transportadora/dia usando transação
                await runTransaction(db, async (transaction) => {
                    const { start, end } = getDayRange(dateInput.value);
                    const q = query(collection(db, "cd_shared_queue_v1"), where("timestamp", ">=", start), where("timestamp", "<", end), where("company", "==", company));
                    const snapshot = await getDocs(q);
                    const nextQueueNumber = snapshot.size + 1;
                    const newEntryRef = doc(collection(db, "cd_shared_queue_v1"));
                    transaction.set(newEntryRef, {
                        driverName: driver,
                        company,
                        packageCount: pkgCount,
                        queueNumber: nextQueueNumber,
                        status: 'waiting',
                        timestamp: Timestamp.now(),
                        completedAt: null
                    });
                    showModal("Sucesso!", `Senha #${nextQueueNumber} registada para ${company}.`, { queueNumber: nextQueueNumber });
                });
                entryForm.reset();
            } catch (error) {
                showModal("Erro", error.message || "Não foi possível registar a entrada.");
            } finally {
                entryButtonText.classList.remove('hidden');
                entrySpinner.classList.add('hidden');
            }
        });

        // Meta diária
        savePlannedPackagesBtn.addEventListener('click', () => {
            plannedPackages = parseInt(plannedPackagesInput.value, 10) || 0;
            localStorage.setItem('plannedPackages_' + dateInput.value, plannedPackages);
            showModal("Sucesso", "Meta diária atualizada!");
            renderCharts(allEntries);
        });

        // Exportação CSV
        exportCsvBtn.addEventListener('click', () => {
            exportButtonText.classList.add('hidden');
            exportSpinner.classList.remove('hidden');
            const entries = allEntries;
            if (entries.length === 0) {
                showModal('Aviso', 'Não há dados para exportar.');
                exportButtonText.classList.remove('hidden');
                exportSpinner.classList.add('hidden');
                return;
            }
            let csvContent = "Relatório de Operações - " + new Date().toLocaleDateString('pt-BR') + "\n\n";
            const waitingEntries = entries.filter(e => e.status === 'waiting');
            const servicedEntries = entries.filter(e => e.status === 'serviced');
            const totalPackages = d3.sum(entries, e => e.packageCount);
            let avgWaitTimeMinutes = '--';
            if (servicedEntries.length > 0) {
                const totalWaitTime = d3.sum(servicedEntries, e => e.completedAt.toDate() - e.timestamp.toDate());
                avgWaitTimeMinutes = Math.round((totalWaitTime / servicedEntries.length) / 60000);
            }
            csvContent += "Resumo do Dia\n";
            csvContent += `Total de Pacotes Recebidos,"${totalPackages.toLocaleString('pt-BR')}"\n`;
            csvContent += `Veículos em Fila,"${waitingEntries.length}"\n`;
            csvContent += `Tempo Médio de Espera (min),"${avgWaitTimeMinutes}"\n\n`;
            csvContent += "Senha,Motorista,Transportadora,Pacotes,Status,Hora Chegada,Hora Atendimento,Tempo de Espera (min)\n";
            entries.sort((a,b) => a.timestamp.toDate() - b.timestamp.toDate());
            entries.forEach(e => {
                const waitTime = e.completedAt ? Math.round((e.completedAt.toDate() - e.timestamp.toDate()) / 60000) : 'N/A';
                const row = [e.queueNumber || 'N/A', e.driverName, e.company, e.packageCount, e.status, e.timestamp.toDate().toLocaleString('pt-BR'), e.completedAt ? e.completedAt.toDate().toLocaleString('pt-BR') : 'N/A', waitTime].map(val => `"${String(val).replace(/"/g, '""')}"`).join(',');
                csvContent += row + '\n';
            });
            const selected = dateInput.value ? new Date(dateInput.value + 'T00:00:00') : new Date();
            const yyyy = selected.getFullYear();
            const mm = String(selected.getMonth() + 1).padStart(2, '0');
            const dd = String(selected.getDate()).padStart(2, '0');
            const blob = new Blob([`\uFEFF${csvContent}`], { type: 'text/csv;charset=utf-8;' });
            const link = document.createElement("a");
            const url = URL.createObjectURL(blob);
            link.setAttribute("href", url);
            link.setAttribute("download", `relatorio_logistica_${yyyy}-${mm}-${dd}.csv`);
            link.style.visibility = 'hidden';
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
            exportButtonText.classList.remove('hidden');
            exportSpinner.classList.add('hidden');
        });

        // Reset do dia
        resetDayBtn.addEventListener('click', async () => {
            const confirmed = await showModal("Confirmar Ação", "Tem a certeza de que deseja apagar todos os registos do dia selecionado? Esta ação é irreversível.", { isConfirm: true });
            if (!confirmed) return;
            resetButtonText.classList.add('hidden');
            resetSpinner.classList.remove('hidden');
            try {
                const { start, end } = getDayRange(dateInput.value);
                const q = query(collection(db, "cd_shared_queue_v1"), where("timestamp", ">=", start), where("timestamp", "<", end));
                const snapshot = await getDocs(q);
                const batch = writeBatch(db);
                snapshot.forEach(docSnap => {
                    batch.delete(docSnap.ref);
                });
                await batch.commit();
                localStorage.removeItem('plannedPackages_' + dateInput.value);
                plannedPackagesInput.value = 0;
                plannedPackages = 0;
                showModal("Sucesso", "Todos os dados do dia selecionado foram apagados.");
            } catch (error) {
                showModal("Erro", "Não foi possível apagar os dados.");
            } finally {
                resetButtonText.classList.remove('hidden');
                resetSpinner.classList.add('hidden');
            }
        });

        // Inicialização de autenticação e listeners
        onAuthStateChanged(auth, user => {
            if (user) {
                loader.classList.add('hidden');
                content.classList.remove('hidden');
                const today = new Date();
                dateInput.value = today.toISOString().slice(0,10);
                plannedPackages = parseInt(localStorage.getItem('plannedPackages_' + dateInput.value) || '0', 10);
                plannedPackagesInput.value = plannedPackages;
                listenForDay(dateInput.value);
            } else {
                signInAnonymously(auth);
            }
        });

        // Atualiza ao trocar a data
        dateInput.addEventListener('change', () => {
            plannedPackages = parseInt(localStorage.getItem('plannedPackages_' + dateInput.value) || '0', 10);
            plannedPackagesInput.value = plannedPackages;
            listenForDay(dateInput.value);
        });

        // Fecha modal ao clicar fora
        window.onclick = function(event) {
            if (event.target === modal) modal.style.display = "none";
