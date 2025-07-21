<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Painel Financeiro - Black Salon</title>
    
    <!-- Tailwind CSS for styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- Chart.js for data visualization -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    
    <!-- Lucide Icons for UI icons -->
    <script src="https://unpkg.com/lucide@latest"></script>

    <!-- html2pdf.js for PDF exporting -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js" integrity="sha512-GsLlZN/3F2ErC5ifS5QtgpiJtWd43JWSuIgh7mbzZ8zBps+dvLusV+eNQATqgA/HdeKFVgA5v3S/cIrLF7QnIg==" crossorigin="anonymous" referrerpolicy="no-referrer"></script>

    <!-- Google Fonts: Inter -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;900&display=swap" rel="stylesheet">

    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #0f172a; /* slate-900 */
            color: #e2e8f0; /* slate-200 */
        }
        #loading-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(15, 23, 42, 0.9);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 9999;
            transition: opacity 0.3s;
            text-align: center;
            padding: 1rem;
            color: white;
        }
        .loader {
            width: 48px;
            height: 48px;
            border: 5px solid #FFF;
            border-bottom-color: #f59e0b; /* amber-500 */
            border-radius: 50%;
            display: inline-block;
            box-sizing: border-box;
            animation: rotation 1s linear infinite;
        }
        @keyframes rotation {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        /* Estilos para o relatório em PDF */
        .pdf-report-container {
            padding: 20px;
            color: #111827; /* Cor do texto escuro para melhor leitura no branco */
        }
        .pdf-report-container h1 {
            font-size: 24px;
            font-weight: bold;
            margin-bottom: 8px;
            color: #000;
        }
        .pdf-report-container p {
            font-size: 14px;
            margin-bottom: 16px;
        }
        .pdf-report-container table {
            width: 100%;
            border-collapse: collapse;
            font-size: 12px;
        }
        .pdf-report-container th, .pdf-report-container td {
            border: 1px solid #e5e7eb;
            padding: 8px;
            text-align: left;
        }
        .pdf-report-container th {
            background-color: #f3f4f6;
        }
    </style>
</head>
<body class="antialiased">

    <div id="loading-overlay">
        <div class="loader-container">
            <div class="loader"></div>
            <p class="mt-4 text-slate-300">A iniciar aplicação...</p>
        </div>
    </div>

    <!-- ================================================================================= -->
    <!-- ÁREA DE AUTENTICAÇÃO (LOGIN / CADASTRO) -->
    <!-- ================================================================================= -->
    <div id="auth-container" class="min-h-screen flex items-center justify-center bg-slate-900">
        <div class="w-full max-w-md p-8 space-y-8 bg-slate-800 rounded-2xl shadow-lg">
            
            <!-- Formulário de Login -->
            <form id="login-form" class="space-y-6">
                <div>
                    <h2 class="text-2xl font-bold text-center text-slate-100">Login no Painel</h2>
                    <p class="text-center text-slate-400">Insira suas credenciais para aceder.</p>
                </div>
                <div>
                    <label for="login-email" class="text-sm font-medium text-slate-400 block mb-1">Email</label>
                    <input type="email" id="login-email" required class="w-full bg-slate-700 border border-slate-600 text-slate-200 rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-amber-500 focus:border-amber-500">
                </div>
                <div>
                    <label for="login-password" class="text-sm font-medium text-slate-400 block mb-1">Senha</label>
                    <input type="password" id="login-password" required class="w-full bg-slate-700 border border-slate-600 text-slate-200 rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-amber-500 focus:border-amber-500">
                </div>
                <p id="login-error" class="text-sm text-red-400 text-center"></p>
                <button type="submit" class="w-full bg-amber-500 text-slate-900 font-bold py-3 rounded-md hover:bg-amber-400 transition-colors flex items-center justify-center gap-2">
                    Entrar
                </button>
                <p class="text-sm text-center text-slate-400">
                    Não tem uma conta? 
                    <a href="#" id="show-register-link" class="font-medium text-amber-400 hover:underline">Cadastre-se</a>
                </p>
            </form>

            <!-- Formulário de Cadastro (inicialmente oculto) -->
            <form id="register-form" class="space-y-6 hidden">
                <div>
                    <h2 class="text-2xl font-bold text-center text-slate-100">Criar Conta</h2>
                    <p class="text-center text-slate-400">Crie seu acesso ao painel financeiro.</p>
                </div>
                <div>
                    <label for="register-email" class="text-sm font-medium text-slate-400 block mb-1">Email</label>
                    <input type="email" id="register-email" required class="w-full bg-slate-700 border border-slate-600 text-slate-200 rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-amber-500 focus:border-amber-500">
                </div>
                <div>
                    <label for="register-password" class="text-sm font-medium text-slate-400 block mb-1">Senha (mínimo 6 caracteres)</label>
                    <input type="password" id="register-password" required class="w-full bg-slate-700 border border-slate-600 text-slate-200 rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-amber-500 focus:border-amber-500">
                </div>
                <p id="register-error" class="text-sm text-red-400 text-center"></p>
                <button type="submit" class="w-full bg-amber-500 text-slate-900 font-bold py-3 rounded-md hover:bg-amber-400 transition-colors flex items-center justify-center gap-2">
                    Cadastrar
                </button>
                <p class="text-sm text-center text-slate-400">
                    Já tem uma conta? 
                    <a href="#" id="show-login-link" class="font-medium text-amber-400 hover:underline">Faça Login</a>
                </p>
            </form>

        </div>
    </div>


    <!-- ================================================================================= -->
    <!-- CONTAINER PRINCIPAL DA APLICAÇÃO (INICIALMENTE OCULTO) -->
    <!-- ================================================================================= -->
    <div id="app-container" class="hidden">
        <div class="container mx-auto p-4 md:p-8">
            <!-- Header Section -->
            <header class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-8">
                <div>
                    <h1 class="text-3xl font-bold text-slate-100">Painel Financeiro</h1>
                    <div class="text-slate-400 flex items-center gap-4 mt-1">
                        <span id="user-email" class="text-sm"></span>
                        <button id="logout-btn" class="text-xs text-amber-400 hover:text-amber-300 hover:underline flex items-center gap-1">
                            <i data-lucide="log-out" class="w-3 h-3"></i>
                            Sair
                        </button>
                    </div>
                </div>
                <div class="mt-4 sm:mt-0 flex items-center gap-2 bg-slate-800 p-1 rounded-lg">
                    <button id="unit1-btn" class="px-4 py-2 text-sm font-bold rounded-md transition-colors text-slate-300 hover:bg-slate-700">Unidade 01</button>
                    <button id="unit2-btn" class="px-4 py-2 text-sm font-bold rounded-md transition-colors text-slate-300 hover:bg-slate-700">Unidade 02</button>
                </div>
            </header>

            <main class="space-y-8">
                
                <section>
                    <h2 id="kpi-title" class="text-xl font-bold text-slate-200 mb-4">Resumo Anual</h2>
                    <div id="kpi-cards-container" class="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-4 gap-6"></div>
                </section>

                <section id="monthly-result-container"></section>

                <section class="grid grid-cols-1 xl:grid-cols-5 gap-8">
                    <div class="bg-slate-800 p-6 rounded-2xl shadow-lg xl:col-span-3">
                        <h3 class="text-lg font-bold text-slate-200 mb-4">Receitas vs Despesas (Anual)</h3>
                        <div class="h-80 relative"><canvas id="bar-chart"></canvas></div>
                    </div>
                    <div class="bg-slate-800 p-6 rounded-2xl shadow-lg xl:col-span-2">
                        <div class="flex justify-between items-center mb-4">
                            <h3 class="text-lg font-bold text-slate-200">Despesas por Categoria</h3>
                            <select id="pie-chart-month-filter" class="bg-slate-700 text-sm text-slate-300 rounded-md p-2 border border-slate-600 focus:ring-2 focus:ring-amber-500 focus:outline-none"></select>
                        </div>
                        <div class="h-80 relative flex items-center justify-center"><canvas id="pie-chart"></canvas></div>
                    </div>
                </section>

                <section id="add-transaction-section" class="bg-slate-800 p-6 rounded-2xl shadow-lg">
                    <h3 class="text-lg font-bold text-slate-200 mb-4">Adicionar Lançamento</h3>
                    
                    <div id="operation-context" class="bg-slate-900/70 p-3 rounded-lg mb-4 text-center">
                        <p class="text-sm text-slate-400">
                            Lançando em: 
                            <span id="current-unit-display" class="font-bold text-amber-400"></span> | 
                            <span id="current-month-display" class="font-bold text-slate-200"></span>
                        </p>
                    </div>
                    
                    <form id="transaction-form" class="space-y-4">
                        <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                            <div>
                                <label for="date" class="text-sm font-medium text-slate-400 block mb-1">Data</label>
                                <input type="date" id="date" required class="w-full bg-slate-700 border border-slate-600 text-slate-200 rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-amber-500">
                            </div>
                            <div>
                                <label for="amount" class="text-sm font-medium text-slate-400 block mb-1">Valor (R$)</label>
                                <input type="number" step="0.01" id="amount" placeholder="0.00" required class="w-full bg-slate-700 border border-slate-600 text-slate-200 rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-amber-500">
                            </div>
                        </div>
                        <div>
                            <label for="description" class="text-sm font-medium text-slate-400 block mb-1">Descrição</label>
                            <input type="text" id="description" placeholder="Ex: Compra de material" required class="w-full bg-slate-700 border border-slate-600 text-slate-200 rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-amber-500">
                        </div>
                        <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                            <div>
                                <label for="type" class="text-sm font-medium text-slate-400 block mb-1">Tipo</label>
                                <select id="type" class="w-full bg-slate-700 border border-slate-600 text-slate-200 rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-amber-500">
                                    <option value="expense">Despesa</option>
                                    <option value="revenue">Receita</option>
                                </select>
                            </div>
                             <div>
                                <label for="status" class="text-sm font-medium text-slate-400 block mb-1">Status</label>
                                <select id="status" class="w-full bg-slate-700 border border-slate-600 text-slate-200 rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-amber-500">
                                    <option value="pago">Pago</option>
                                    <option value="aberto">Aberto</option>
                                </select>
                            </div>
                        </div>
                        <div>
                            <label for="category" class="text-sm font-medium text-slate-400 block mb-1">Categoria</label>
                            <select id="category" class="w-full bg-slate-700 border border-slate-600 text-slate-200 rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-amber-500"></select>
                        </div>
                        <button type="submit" class="w-full bg-amber-500 text-slate-900 font-bold py-3 rounded-md hover:bg-amber-400 transition-colors flex items-center justify-center gap-2">
                            <i data-lucide="plus-circle" class="w-5 h-5"></i>
                            Adicionar
                        </button>
                    </form>
                </section>

                <section id="transaction-list-section" class="bg-slate-800 p-6 rounded-2xl shadow-lg">
                    <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-4 gap-4">
                        <h3 class="text-lg font-bold text-slate-200">Últimos Lançamentos</h3>
                        
                        <div class="flex items-center gap-2 w-full sm:w-auto">
                            <select id="month-filter" class="flex-grow sm:flex-grow-0 bg-slate-700 text-sm text-slate-300 rounded-md p-2 border border-slate-600 focus:outline-none focus:ring-2 focus:ring-amber-500"></select>
                            <select id="category-filter" class="flex-grow sm:flex-grow-0 bg-slate-700 text-sm text-slate-300 rounded-md p-2 border border-slate-600 focus:outline-none focus:ring-2 focus:ring-amber-500"></select>
                            
                            <button id="export-csv-btn" title="Exportar para CSV" class="text-slate-400 hover:text-amber-400 p-2 rounded-md bg-slate-700 hover:bg-slate-600 transition">
                                <i data-lucide="file-spreadsheet" class="w-5 h-5"></i>
                            </button>
                            <button id="export-pdf-btn" title="Exportar para PDF" class="text-slate-400 hover:text-amber-400 p-2 rounded-md bg-slate-700 hover:bg-slate-600 transition">
                                <i data-lucide="file-text" class="w-5 h-5"></i>
                            </button>
                        </div>
                    </div>
                    <div class="overflow-x-auto">
                        <table class="w-full text-sm text-left text-slate-400">
                            <thead class="text-xs text-slate-400 uppercase bg-slate-900/50">
                                <tr>
                                    <th scope="col" class="px-4 py-3">Data</th>
                                    <th scope="col" class="px-4 py-3">Descrição</th>
                                    <th scope="col" class="px-4 py-3">Categoria</th>
                                    <th scope="col" class="px-4 py-3 text-center">Status</th>
                                    <th scope="col" class="px-4 py-3 text-right">Valor</th>
                                    <th scope="col" class="px-4 py-3 text-center">Ação</th>
                                </tr>
                            </thead>
                            <tbody id="transaction-table-body"></tbody>
                        </table>
                    </div>
                </section>
            </main>
        </div>
    </div>


    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, onAuthStateChanged, createUserWithEmailAndPassword, signInWithEmailAndPassword, signOut } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, onSnapshot, updateDoc, deleteField } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        document.addEventListener('DOMContentLoaded', () => {
            
            // CREDENCIAIS DO FIREBASE FORNECIDAS PELO USUÁRIO
            const firebaseConfig = {
              apiKey: "AIzaSyDPVEZrAdQDibsOKVNNBWnZN8SqVKOvV4E",
              authDomain: "gestao-financeira-black-salon.firebaseapp.com",
              projectId: "gestao-financeira-black-salon",
              storageBucket: "gestao-financeira-black-salon.appspot.com",
              messagingSenderId: "972714970450",
              appId: "1:972714970450:web:768f06ee2a82c28b36be8b",
              measurementId: "G-40VVV9HJHK"
            };

            const loadingOverlay = document.getElementById('loading-overlay');
            const authContainer = document.getElementById('auth-container');
            const appContainer = document.getElementById('app-container');
            const loginForm = document.getElementById('login-form');
            const registerForm = document.getElementById('register-form');
            const logoutBtn = document.getElementById('logout-btn');
            const showRegisterLink = document.getElementById('show-register-link');
            const showLoginLink = document.getElementById('show-login-link');
            const userEmailDisplay = document.getElementById('user-email');
            
            // Verificação de sanidade para garantir que a chave foi inserida
            if (!firebaseConfig.apiKey || !firebaseConfig.apiKey.startsWith("AIza")) {
                loadingOverlay.innerHTML = `
                    <div class="text-center p-4">
                        <h2 class="text-xl font-bold text-red-400 mb-2">Erro de Configuração</h2>
                        <p class="text-slate-300">A configuração do Firebase parece estar incorreta ou ausente.<br>Por favor, verifique as credenciais no código.</p>
                    </div>`;
                return;
            }

            const app = initializeApp(firebaseConfig);
            const auth = getAuth(app);
            const db = getFirestore(app);

            let userId = null;
            let unsubscribeFromData;
            let isAppInitialized = false;

            let allData = { unit1: { transactions: [], aportes: {} }, unit2: { transactions: [], aportes: {} } };
            let activeUnit = 'unit1';
            let barChartInstance;
            let pieChartInstance;
            
            const revenueCategories = ['Venda de Assinaturas', 'Outras receitas', 'Rendimento Financeiro', 'Venda de produto', 'Venda de serviço', 'Venda de pacote'];
            const expenseCategories = ['Comissão sobre serviços', 'Água', 'Aluguel', 'App booksy', 'Internet', 'Luz', 'Material de adm', 'Pró-labore', 'Tv a cabo', 'Contador', 'Marketing', 'Wix site', 'Chatgpt', 'Youtube', 'Salário', 'Automação(make,zapi,gpt)', 'Almoço', 'Manutenção e reformas', 'Copa/bebedouro', 'Moveis e Equipamentos', 'INSS', 'FGTS', 'DAS', 'Custo do produto vendido', 'Juros e multas', 'Outros Impostos', 'Tarifas Bancárias', 'Taxa cartao', 'Outras despesas', 'Custo dos produtos utilizados', 'Taxa cartão assinatura', 'Aluguel maq cartao', 'Tarifa pix qrcode'];
            const months = ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun', 'Jul', 'Ago', 'Set', 'Out', 'Nov', 'Dez'];
            const monthNames = ['Janeiro', 'Fevereiro', 'Março', 'Abril', 'Maio', 'Junho', 'Julho', 'Agosto', 'Setembro', 'Outubro', 'Novembro', 'Dezembro'];
            const PIE_COLORS = ['#f59e0b', '#10b981', '#3b82f6', '#ef4444', '#8b5cf6', '#ec4899', '#6366f1', '#d946ef', '#06b6d4', '#f97316', '#f43f5e', '#84cc16', '#22d3ee', '#a855f7'];

            const addTransactionToDB = async (transaction) => { if (!userId) return; const docRef = doc(db, `financial_data/${userId}/units/${activeUnit}`); await setDoc(docRef, { transactions: { [transaction.id]: transaction } }, { merge: true }); };
            const updateTransactionInDB = async (id, field, value) => { if (!userId) return; const docRef = doc(db, `financial_data/${userId}/units/${activeUnit}`); const fieldPath = `transactions.${id}.${field}`; await updateDoc(docRef, { [fieldPath]: value }); };
            const deleteTransactionFromDB = async (id) => { if (!userId) return; const docRef = doc(db, `financial_data/${userId}/units/${activeUnit}`); const fieldPath = `transactions.${id}`; await updateDoc(docRef, { [fieldPath]: deleteField() }); };
            const updateAporteInDB = async (monthIndex, amount) => { if (!userId) return; const docRef = doc(db, `financial_data/${userId}/units/${activeUnit}`); await setDoc(docRef, { aportes: { [String(monthIndex)]: amount } }, { merge: true }); };

            const formatCurrency = (value) => new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(value);
            const getAnnualData = (transactions) => { const paidTransactions = transactions.filter(t => t.status === 'pago'); return months.map((month, index) => { const monthTransactions = paidTransactions.filter(t => new Date(t.date).getUTCMonth() === index); const revenue = monthTransactions.filter(t => t.type === 'revenue').reduce((acc, t) => acc + t.amount, 0); const expenses = monthTransactions.filter(t => t.type === 'expense').reduce((acc, t) => acc + t.amount, 0); return { month, revenue, expenses, profit: revenue - expenses }; }); };
            const getTotals = (annualData) => { const totalRevenue = annualData.reduce((acc, item) => acc + item.revenue, 0); const totalExpenses = annualData.reduce((acc, item) => acc + item.expenses, 0); const netProfit = totalRevenue - totalExpenses; const profitMargin = totalRevenue > 0 ? (netProfit / totalRevenue) * 100 : 0; return { totalRevenue, totalExpenses, netProfit, profitMargin }; };
            const getPieChartData = (transactions, monthIndex) => { const paidExpenses = transactions.filter(t => new Date(t.date).getUTCMonth() === monthIndex && t.type === 'expense' && t.status === 'pago'); const expensesByCategory = paidExpenses.reduce((acc, t) => { if (!acc[t.category]) acc[t.category] = 0; acc[t.category] += t.amount; return acc; }, {}); return Object.entries(expensesByCategory).map(([name, value]) => ({ name, value })); };
            const getMonthlySummary = (transactions, monthlyAportes, selectedMonthIndex) => { const getMonthCashFlowResult = (monthIdx) => { const monthTransactions = transactions.filter(t => new Date(t.date).getUTCMonth() === monthIdx && t.status === 'pago'); const revenue = monthTransactions.filter(t => t.type === 'revenue').reduce((acc, t) => acc + t.amount, 0); const expenses = monthTransactions.filter(t => t.type === 'expense').reduce((acc, t) => acc + t.amount, 0); const aporte = monthlyAportes[monthIdx] || 0; return revenue - expenses + aporte; }; let accumulatedResult = 0; for (let i = 0; i < selectedMonthIndex; i++) { accumulatedResult += getMonthCashFlowResult(i); } const saldoAnteriorRealizado = accumulatedResult; const monthTransactions = transactions.filter(t => new Date(t.date).getUTCMonth() === selectedMonthIndex); const receitasPagas = monthTransactions.filter(t => t.type === 'revenue' && t.status === 'pago').reduce((acc, t) => acc + t.amount, 0); const despesasPagas = monthTransactions.filter(t => t.type === 'expense' && t.status === 'pago').reduce((acc, t) => acc + t.amount, 0); const aporteDoMes = monthlyAportes[selectedMonthIndex] || 0; const resultadoOperacional = receitasPagas - despesasPagas; const saldoAtualRealizado = saldoAnteriorRealizado + resultadoOperacional + aporteDoMes; const aReceber = monthTransactions.filter(t => t.type === 'revenue' && t.status === 'aberto').reduce((acc, t) => acc + t.amount, 0); const aPagar = monthTransactions.filter(t => t.type === 'expense' && t.status === 'aberto').reduce((acc, t) => acc + t.amount, 0); const saldoPrevisto = saldoAtualRealizado + aReceber - aPagar; return { saldoAnteriorRealizado, receitasPagas, despesasPagas, aporteDoMes, resultadoOperacional, saldoAtualRealizado, aReceber, aPagar, saldoPrevisto }; };

            const updateOperationContext = () => { const unitDisplay = document.getElementById('current-unit-display'); const monthDisplay = document.getElementById('current-month-display'); const dateInput = document.getElementById('date'); if (unitDisplay && monthDisplay && dateInput) { unitDisplay.textContent = activeUnit === 'unit1' ? 'Unidade 01' : 'Unidade 02'; const selectedDate = new Date(dateInput.value + 'T12:00:00Z'); const monthName = monthNames[selectedDate.getUTCMonth()]; const year = selectedDate.getUTCFullYear(); monthDisplay.textContent = `${monthName} de ${year}`; } };
            const renderUnitButtons = () => { const unit1Btn = document.getElementById('unit1-btn'); const unit2Btn = document.getElementById('unit2-btn'); if (activeUnit === 'unit1') { unit1Btn.className = 'px-4 py-2 text-sm font-bold rounded-md transition-colors bg-amber-500 text-slate-900'; unit2Btn.className = 'px-4 py-2 text-sm font-bold rounded-md transition-colors text-slate-300 hover:bg-slate-700'; } else { unit1Btn.className = 'px-4 py-2 text-sm font-bold rounded-md transition-colors text-slate-300 hover:bg-slate-700'; unit2Btn.className = 'px-4 py-2 text-sm font-bold rounded-md transition-colors bg-amber-500 text-slate-900'; } };
            const renderKpiCards = () => { const container = document.getElementById('kpi-cards-container'); document.getElementById('kpi-title').textContent = `Resumo Anual - ${activeUnit === 'unit1' ? 'Unidade 01' : 'Unidade 02'}`; const annualData = getAnnualData(allData[activeUnit].transactions); const totals = getTotals(annualData); const cards = [ { title: "Receita Realizada (Anual)", value: formatCurrency(totals.totalRevenue), icon: "dollar-sign", color: "bg-blue-500" }, { title: "Despesa Realizada (Anual)", value: formatCurrency(totals.totalExpenses), icon: "trending-down", color: "bg-red-500" }, { title: "Lucro Operacional (Anual)", value: formatCurrency(totals.netProfit), icon: "trending-up", color: "bg-green-500" }, { title: "Margem de Lucro", value: `${totals.profitMargin.toFixed(2)}%`, icon: "percent", color: "bg-amber-500" } ]; container.innerHTML = cards.map(card => `<div class="bg-slate-800 p-6 rounded-2xl shadow-lg flex items-center justify-between transition-all duration-300 hover:shadow-amber-500/20 hover:scale-[1.03]"> <div> <p class="text-sm font-medium text-slate-400">${card.title}</p> <p class="text-2xl font-bold text-slate-100">${card.value}</p> </div> <div class="p-3 rounded-full ${card.color} shadow-lg"> <i data-lucide="${card.icon}" class="h-6 w-6 text-white"></i> </div> </div>`).join(''); };
            const renderMonthlyResult = () => { const container = document.getElementById('monthly-result-container'); const selectedMonthIndex = parseInt(container.querySelector('#month-summary-select')?.value || new Date().getMonth()); const summary = getMonthlySummary(allData[activeUnit].transactions, allData[activeUnit].aportes, selectedMonthIndex); container.innerHTML = `<div class="bg-slate-800 rounded-2xl shadow-lg p-6"> <div class="flex flex-col md:flex-row justify-between items-center mb-6"> <h3 class="text-lg font-bold text-slate-200">Demonstrativo de Fluxo de Caixa</h3> <select id="month-summary-select" class="bg-slate-700 text-md font-bold text-amber-400 focus:outline-none focus:ring-2 focus:ring-amber-500 appearance-none text-center p-2 rounded-lg mt-2 md:mt-0"> ${monthNames.map((name, index) => `<option value="${index}" ${index === selectedMonthIndex ? 'selected' : ''}>${name}</option>`).join('')} </select> </div> <div class="grid grid-cols-2 md:grid-cols-3 gap-4 text-center"> <div class="p-3 bg-slate-900/50 rounded-lg"><p class="text-sm font-medium text-slate-400">Saldo Anterior</p><p class="text-lg font-bold ${summary.saldoAnteriorRealizado >= 0 ? 'text-slate-200' : 'text-red-400'}">${formatCurrency(summary.saldoAnteriorRealizado)}</p></div> <div class="p-3 bg-green-900/20 rounded-lg"><p class="text-sm font-medium text-green-400">Receitas Pagas</p><p class="text-lg font-bold text-slate-200">${formatCurrency(summary.receitasPagas)}</p></div> <div class="p-3 bg-red-900/20 rounded-lg"><p class="text-sm font-medium text-red-400">Despesas Pagas</p><p class="text-lg font-bold text-slate-200">${formatCurrency(summary.despesasPagas)}</p></div> <div class="p-3 bg-blue-900/20 rounded-lg"> <p class="text-sm font-medium text-blue-400 flex items-center justify-center gap-1">Aporte do Mês <i data-lucide="edit" class="w-3 h-3"></i></p> <input id="aporte-input" type="number" step="0.01" data-month-index="${selectedMonthIndex}" value="${summary.aporteDoMes || ''}" class="text-lg font-bold bg-transparent text-center w-full focus:outline-none p-0 border-0 text-slate-200" placeholder="R$ 0,00"/> </div> <div class="p-3 bg-slate-900/50 rounded-lg"><p class="text-sm font-medium text-slate-400">Resultado Operacional</p><p class="text-lg font-bold ${summary.resultadoOperacional >= 0 ? 'text-blue-400' : 'text-red-400'}">${formatCurrency(summary.resultadoOperacional)}</p></div> <div class="p-3 bg-amber-900/30 rounded-lg"><p class="text-sm font-medium text-amber-400">Saldo Atual</p><p class="text-lg font-bold ${summary.saldoAtualRealizado >= 0 ? 'text-slate-200' : 'text-red-400'}">${formatCurrency(summary.saldoAtualRealizado)}</p></div> <div class="p-3 bg-green-900/20 rounded-lg mt-4 col-span-1"><p class="text-sm font-medium text-green-400">A Receber</p><p class="text-lg font-bold text-slate-200">${formatCurrency(summary.aReceber)}</p></div> <div class="p-3 bg-red-900/20 rounded-lg mt-4 col-span-1"><p class="text-sm font-medium text-red-400">A Pagar</p><p class="text-lg font-bold text-slate-200">${formatCurrency(summary.aPagar)}</p></div> <div class="p-3 bg-slate-900/50 rounded-lg mt-4 col-span-2 md:col-span-1"><p class="text-sm font-medium text-slate-400">Saldo Previsto</p><p class="text-lg font-bold ${summary.saldoPrevisto >= 0 ? 'text-slate-200' : 'text-red-400'}">${formatCurrency(summary.saldoPrevisto)}</p></div> </div> </div>`; };
            
            const populateCategoryFilter = () => {
                const categoryFilter = document.getElementById('category-filter');
                if (!categoryFilter) return;
                const transactions = allData[activeUnit].transactions;
                const categories = [...new Set(transactions.map(t => t.category))].sort();
                const currentValue = categoryFilter.value;
                categoryFilter.innerHTML = '<option value="all">Todas as Categorias</option>' + categories.map(cat => `<option value="${cat}">${cat}</option>`).join('');
                if (categories.includes(currentValue)) {
                    categoryFilter.value = currentValue;
                }
            };
            
            const getFilteredTransactions = () => {
                const filterMonth = document.getElementById('month-filter').value;
                const filterCategory = document.getElementById('category-filter').value;
                const transactions = allData[activeUnit].transactions;
                const sortedTransactions = [...transactions].sort((a, b) => new Date(b.date) - new Date(a.date));
                let filteredTransactions = sortedTransactions;
                
                if (filterCategory !== 'all') {
                    filteredTransactions = filteredTransactions.filter(t => t.category === filterCategory);
                }
                
                if (filterMonth !== 'all') {
                    filteredTransactions = filteredTransactions.filter(t => new Date(t.date).getUTCMonth() === parseInt(filterMonth));
                }
                return filteredTransactions;
            };

            const getTransactionsForExport = () => {
                // Função similar a getFilteredTransactions, mas sem o slice(15) para garantir que todos os dados sejam exportados
                const filterMonth = document.getElementById('month-filter').value;
                const filterCategory = document.getElementById('category-filter').value;
                const transactions = allData[activeUnit].transactions;
                const sortedTransactions = [...transactions].sort((a, b) => new Date(b.date) - new Date(a.date));
                let filteredTransactions = sortedTransactions;
                
                if (filterCategory !== 'all') {
                    filteredTransactions = filteredTransactions.filter(t => t.category === filterCategory);
                }
                
                if (filterMonth !== 'all') {
                    filteredTransactions = filteredTransactions.filter(t => new Date(t.date).getUTCMonth() === parseInt(filterMonth));
                }
                
                return filteredTransactions;
            }

            const renderTransactionList = () => {
                const tbody = document.getElementById('transaction-table-body');
                let transactionsToDisplay = getFilteredTransactions();
                const filterMonth = document.getElementById('month-filter').value;

                // Aplica o slice de 15 apenas se o filtro de mês for "todos" para a visualização na tela
                if (filterMonth === 'all') {
                    transactionsToDisplay = transactionsToDisplay.slice(0, 15);
                }

                if (transactionsToDisplay.length === 0) { tbody.innerHTML = `<tr><td colspan="6" class="text-center py-8 text-slate-500">Nenhum lançamento encontrado para os filtros selecionados.</td></tr>`; return; }
                tbody.innerHTML = transactionsToDisplay.map(t => `
                    <tr class="border-b border-slate-700 hover:bg-slate-700/50">
                        <td class="px-4 py-4 whitespace-nowrap">${new Date(t.date).toLocaleDateString('pt-BR', {timeZone: 'UTC'})}</td>
                        <td class="px-4 py-4 font-medium text-slate-200 max-w-xs truncate">
                            <input type="text" value="${t.description.replace(/"/g, '"')}" data-action="update" data-id="${t.id}" data-field="description" class="bg-transparent w-full focus:outline-none focus:bg-slate-700 rounded px-1"/>
                        </td>
                        <td class="px-4 py-4 whitespace-nowrap">${t.category}</td>
                        <td class="px-4 py-4 text-center">
                            <button data-action="toggle-status" data-id="${t.id}" class="text-xs font-semibold inline-flex items-center px-2.5 py-0.5 rounded-full ${t.status === 'pago' ? 'bg-green-500/20 text-green-300 hover:bg-green-500/40' : 'bg-yellow-500/20 text-yellow-300 hover:bg-yellow-500/40'} transition-colors">
                                <i data-lucide="${t.status === 'pago' ? 'check-circle' : 'clock'}" class="w-3 h-3 mr-1.5 pointer-events-none"></i>
                                ${t.status.charAt(0).toUpperCase() + t.status.slice(1)}
                            </button>
                        </td>
                        <td class="px-4 py-4 text-right font-medium whitespace-nowrap">
                            <input type="number" step="0.01" value="${t.amount}" data-action="update" data-id="${t.id}" data-field="amount" class="bg-transparent w-full text-right focus:outline-none focus:bg-slate-700 rounded px-1 ${t.type === 'revenue' ? 'text-green-400' : 'text-red-400'}"/>
                        </td>
                        <td class="px-4 py-4 text-center">
                            <button data-action="delete" data-id="${t.id}" title="Excluir Lançamento" class="text-slate-500 hover:text-red-400 p-1 rounded-md transition-colors">
                                <i data-lucide="trash-2" class="h-4 w-4 pointer-events-none"></i>
                            </button>
                        </td>
                    </tr>`).join('');
            };
            const renderCharts = () => { const transactions = allData[activeUnit].transactions; const annualData = getAnnualData(transactions); const barCtx = document.getElementById('bar-chart').getContext('2d'); if (barChartInstance) barChartInstance.destroy(); barChartInstance = new Chart(barCtx, { type: 'bar', data: { labels: annualData.map(d => d.month), datasets: [ { label: 'Receita', data: annualData.map(d => d.revenue), backgroundColor: '#3b82f6', borderRadius: 4 }, { label: 'Despesa', data: annualData.map(d => d.expenses), backgroundColor: '#ef4444', borderRadius: 4 } ] }, options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { labels: { color: '#94a3b8' } } }, scales: { x: { ticks: { color: '#94a3b8' }, grid: { display: false } }, y: { ticks: { color: '#94a3b8', callback: value => formatCurrency(value) }, grid: { color: 'rgba(255, 255, 255, 0.1)', borderDash: [3, 3] } } } } }); const pieMonthFilter = document.getElementById('pie-chart-month-filter'); const selectedMonthIndex = parseInt(pieMonthFilter.value); const pieData = getPieChartData(transactions, selectedMonthIndex); const pieCtx = document.getElementById('pie-chart').getContext('2d'); if (pieChartInstance) pieChartInstance.destroy(); pieChartInstance = new Chart(pieCtx, { type: 'pie', data: { labels: pieData.map(d => d.name), datasets: [{ data: pieData.map(d => d.value), backgroundColor: PIE_COLORS, borderColor: '#1e293b', hoverOffset: 4 }] }, options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { display: false }, tooltip: { callbacks: { label: (context) => ` ${context.label}: ${formatCurrency(context.raw)}` } } } } }); };
            
            const handleExportPDF = () => {
                const filteredTransactions = getTransactionsForExport();
                if(filteredTransactions.length === 0) {
                    alert("Não há dados para exportar com os filtros selecionados.");
                    return;
                }

                const unitName = activeUnit === 'unit1' ? 'Unidade 01' : 'Unidade 02';
                const monthFilter = document.getElementById('month-filter');
                const selectedMonthName = monthFilter.value === 'all' ? 'Geral (Todos os Meses)' : monthNames[parseInt(monthFilter.value)];
                const categoryFilter = document.getElementById('category-filter');
                const selectedCategoryName = categoryFilter.value === 'all' ? 'Todas as Categorias' : categoryFilter.value;

                let totalRevenue = 0;
                let totalExpense = 0;

                const tableRows = filteredTransactions.map(t => {
                    if (t.type === 'revenue') totalRevenue += t.amount;
                    else totalExpense += t.amount;
                    return `
                        <tr>
                            <td>${new Date(t.date).toLocaleDateString('pt-BR', {timeZone: 'UTC'})}</td>
                            <td>${t.description}</td>
                            <td>${t.category}</td>
                            <td>${t.status}</td>
                            <td style="color: ${t.type === 'revenue' ? 'green' : 'red'}; text-align: right;">${formatCurrency(t.amount)}</td>
                        </tr>
                    `;
                }).join('');

                const reportHtml = `
                    <div class="pdf-report-container">
                        <h1>Relatório de Lançamentos</h1>
                        <p>
                            <strong>Unidade:</strong> ${unitName} <br>
                            <strong>Período:</strong> ${selectedMonthName} <br>
                            <strong>Categoria:</strong> ${selectedCategoryName} <br>
                            <strong>Data de Emissão:</strong> ${new Date().toLocaleDateString('pt-BR')}
                        </p>
                        <table>
                            <thead>
                                <tr>
                                    <th>Data</th>
                                    <th>Descrição</th>
                                    <th>Categoria</th>
                                    <th>Status</th>
                                    <th style="text-align: right;">Valor</th>
                                </tr>
                            </thead>
                            <tbody>
                                ${tableRows}
                            </tbody>
                        </table>
                        <div style="margin-top: 20px; text-align: right;">
                            <p><strong>Total de Receitas:</strong> ${formatCurrency(totalRevenue)}</p>
                            <p><strong>Total de Despesas:</strong> ${formatCurrency(totalExpense)}</p>
                            <h3 style="font-size: 16px;"><strong>Saldo do Período:</strong> ${formatCurrency(totalRevenue - totalExpense)}</h3>
                        </div>
                    </div>
                `;

                const opt = {
                    margin:       0.5,
                    filename:     `relatorio_${unitName.replace(' ', '_')}_${selectedMonthName.split(' ')[0]}.pdf`,
                    image:        { type: 'jpeg', quality: 0.98 },
                    html2canvas:  { scale: 2, useCORS: true },
                    jsPDF:        { unit: 'in', format: 'letter', orientation: 'portrait' }
                };

                html2pdf().from(reportHtml).set(opt).save();
            };
            
            const handleExportCSV = () => {
                const filteredTransactions = getTransactionsForExport();
                if(filteredTransactions.length === 0) {
                    alert("Não há dados para exportar com os filtros selecionados.");
                    return;
                }

                const headers = ['Data', 'Descricao', 'Categoria', 'Status', 'Tipo', 'Valor'];
                let csvContent = headers.join(',') + '\n';

                filteredTransactions.forEach(t => {
                    const description = `"${t.description.replace(/"/g, '""')}"`; 
                    const amount = t.amount.toFixed(2).replace('.', ',');
                    const row = [
                        new Date(t.date).toLocaleDateString('pt-BR', {timeZone: 'UTC'}),
                        description,
                        t.category,
                        t.status,
                        t.type,
                        amount
                    ].join(',');
                    csvContent += row + '\n';
                });
                
                const blob = new Blob(['\uFEFF' + csvContent], { type: 'text/csv;charset=utf-8;' });
                const link = document.createElement("a");
                const url = URL.createObjectURL(blob);
                
                const unitName = activeUnit === 'unit1' ? 'Unidade_01' : 'Unidade_02';
                const monthFilter = document.getElementById('month-filter');
                const monthName = monthFilter.value === 'all' ? 'Geral' : monthNames[parseInt(monthFilter.value)];

                link.setAttribute("href", url);
                link.setAttribute("download", `relatorio_${unitName}_${monthName}.csv`);
                document.body.appendChild(link);
                link.click();
                document.body.removeChild(link);
                URL.revokeObjectURL(url);
            };
            
            const renderApp = () => {
                if (!userId) return;
                populateCategoryFilter();
                renderUnitButtons();
                renderMonthlyResult();
                renderTransactionList();
                renderKpiCards();
                renderCharts();
                updateOperationContext();
                lucide.createIcons();
            };

            const setupAppEventListeners = () => {
                document.getElementById('unit1-btn').addEventListener('click', () => { activeUnit = 'unit1'; listenToData(); });
                document.getElementById('unit2-btn').addEventListener('click', () => { activeUnit = 'unit2'; listenToData(); });
                
                document.getElementById('month-filter').addEventListener('change', renderTransactionList);
                document.getElementById('category-filter').addEventListener('change', renderTransactionList);
                
                document.getElementById('pie-chart-month-filter').addEventListener('change', renderCharts);
                document.getElementById('monthly-result-container').addEventListener('change', (e) => { if (e.target.id === 'month-summary-select') { renderMonthlyResult(); lucide.createIcons(); } });
                document.getElementById('monthly-result-container').addEventListener('blur', (e) => { if (e.target.id === 'aporte-input') { const monthIndex = parseInt(e.target.dataset.monthIndex); const amount = parseFloat(e.target.value) || 0; updateAporteInDB(monthIndex, amount); } }, true);
                const typeSelect = document.getElementById('type');
                const categorySelect = document.getElementById('category');
                typeSelect.addEventListener('change', () => { const categories = typeSelect.value === 'expense' ? expenseCategories : revenueCategories; categorySelect.innerHTML = categories.map(cat => `<option value="${cat}">${cat}</option>`).join(''); });
                document.getElementById('transaction-form').addEventListener('submit', (e) => { e.preventDefault(); const form = e.target; const newTransaction = { id: crypto.randomUUID(), date: form.elements.date.value, description: form.elements.description.value, type: form.elements.type.value, category: form.elements.category.value, amount: parseFloat(form.elements.amount.value), status: form.elements.status.value }; addTransactionToDB(newTransaction); form.reset(); form.elements.date.value = new Date().toISOString().slice(0, 10); typeSelect.dispatchEvent(new Event('change')); });
                document.getElementById('transaction-table-body').addEventListener('click', (e) => { const button = e.target.closest('button'); if (!button || !button.dataset.action) return; const action = button.dataset.action; const id = button.dataset.id; if (action === 'delete') { if (confirm("Tem certeza que deseja excluir este lançamento?")) { deleteTransactionFromDB(id); } } else if (action === 'toggle-status') { const transaction = allData[activeUnit].transactions.find(t => t.id === id); if (transaction) { const newStatus = transaction.status === 'pago' ? 'aberto' : 'pago'; updateTransactionInDB(id, 'status', newStatus); } } });
                document.getElementById('transaction-table-body').addEventListener('change', (e) => { if (e.target.dataset.action === 'update') { const id = e.target.dataset.id; const field = e.target.dataset.field; const value = field === 'amount' ? parseFloat(e.target.value) || 0 : e.target.value; const transaction = allData[activeUnit].transactions.find(t => t.id === id); if (transaction && transaction[field] !== value) { updateTransactionInDB(id, field, value); } } });
                
                document.getElementById('export-csv-btn').addEventListener('click', handleExportCSV);
                document.getElementById('export-pdf-btn').addEventListener('click', handleExportPDF);
                
                document.getElementById('date').addEventListener('change', updateOperationContext);
            };

            const initializeAppUI = () => {
                if(isAppInitialized) return;
                setupAppEventListeners();
                document.getElementById('date').value = new Date().toISOString().slice(0, 10);
                const monthFilter = document.getElementById('month-filter');
                monthFilter.innerHTML = '<option value="all">Todos os Meses</option>' + monthNames.map((name, index) => `<option value="${index}">${name}</option>`).join('');
                const pieMonthFilter = document.getElementById('pie-chart-month-filter');
                const currentMonth = new Date().getMonth();
                pieMonthFilter.innerHTML = monthNames.map((name, index) => `<option value="${index}" ${index === currentMonth ? 'selected' : ''}>${name}</option>`).join('');
                document.getElementById('type').dispatchEvent(new Event('change'));
                isAppInitialized = true;
            };

            const listenToData = () => {
                if (unsubscribeFromData) unsubscribeFromData();
                const docRef = doc(db, `financial_data/${userId}/units/${activeUnit}`);
                unsubscribeFromData = onSnapshot(docRef, (docSnap) => {
                    if (docSnap.exists()) {
                        const data = docSnap.data();
                        const transactionsObject = data.transactions || {};
                        allData[activeUnit] = { transactions: Object.values(transactionsObject), aportes: data.aportes || {} };
                    } else {
                        setDoc(docRef, { transactions: {}, aportes: {} });
                        allData[activeUnit] = { transactions: [], aportes: {} };
                    }
                    renderApp();
                }, (error) => { console.error("Erro ao ouvir dados do Firestore:", error); loadingOverlay.innerHTML = `<div class="p-4 text-center"><h2 class="text-xl font-bold text-red-400">Erro de Conexão</h2><p class="text-slate-300">Não foi possível ler os dados. Verifique sua conexão e as regras de segurança do Firestore.</p><p class="text-xs text-slate-500 mt-2">Detalhe: ${error.message}</p></div>`; });
            };
            
            loginForm.addEventListener('submit', (e) => { e.preventDefault(); const email = loginForm['login-email'].value; const password = loginForm['login-password'].value; const errorP = document.getElementById('login-error'); errorP.textContent = ''; signInWithEmailAndPassword(auth, email, password).catch(error => { errorP.textContent = 'Email ou senha inválidos.'; console.error("Erro de login:", error.message); }); });
            registerForm.addEventListener('submit', (e) => { e.preventDefault(); const email = registerForm['register-email'].value; const password = registerForm['register-password'].value; const errorP = document.getElementById('register-error'); errorP.textContent = ''; createUserWithEmailAndPassword(auth, email, password).catch(error => { if (error.code === 'auth/weak-password') { errorP.textContent = 'A senha deve ter pelo menos 6 caracteres.'; } else if (error.code === 'auth/email-already-in-use') { errorP.textContent = 'Este email já está em uso.'; } else { errorP.textContent = 'Ocorreu um erro ao criar a conta.'; } console.error("Erro de cadastro:", error.message); }); });
            logoutBtn.addEventListener('click', () => { signOut(auth); });
            showRegisterLink.addEventListener('click', (e) => { e.preventDefault(); loginForm.classList.add('hidden'); registerForm.classList.remove('hidden'); });
            showLoginLink.addEventListener('click', (e) => { e.preventDefault(); registerForm.classList.add('hidden'); loginForm.classList.remove('hidden'); });

            onAuthStateChanged(auth, (user) => {
                if (user) {
                    userId = user.uid;
                    userEmailDisplay.textContent = user.email;
                    authContainer.classList.add('hidden');
                    appContainer.classList.remove('hidden');
                    initializeAppUI();
                    listenToData();
                    loadingOverlay.style.opacity = '0';
                    setTimeout(() => loadingOverlay.style.display = 'none', 300);
                } else {
                    userId = null;
                    if (unsubscribeFromData) unsubscribeFromData();
                    authContainer.classList.remove('hidden');
                    appContainer.classList.add('hidden');
                    loadingOverlay.style.opacity = '0';
                    setTimeout(() => loadingOverlay.style.display = 'none', 300);
                }
            });
        });
    </script>
</body>
</html>
