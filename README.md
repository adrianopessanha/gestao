<script>
    document.addEventListener('DOMContentLoaded', () => {

        // =================================================================================
        // ESTADO INICIAL E CONSTANTES (Sem alterações aqui)
        // =================================================================================
        const STORAGE_KEY = 'black_salon_finances_v12';
        let allData;
        let activeUnit;
        let barChartInstance;
        let pieChartInstance;
        
        const revenueCategories = ['Venda de Assinaturas', 'Outras receitas', 'Rendimento Financeiro', 'Venda de produto', 'Venda de serviço', 'Venda de pacote'];
        const expenseCategories = ['Comissão sobre serviços', 'Água', 'Aluguel', 'App booksy', 'Internet', 'Luz', 'Material de adm', 'Pró-labore', 'Tv a cabo', 'Contador', 'Marketing', 'Wix site', 'Chatgpt', 'Youtube', 'Salário', 'Automação(make,zapi,gpt)', 'Almoço', 'Manutenção e reformas', 'Copa/bebedouro', 'Moveis e Equipamentos', 'INSS', 'FGTS', 'DAS', 'Custo do produto vendido', 'Juros e multas', 'Outros Impostos', 'Tarifas Bancárias', 'Taxa cartao', 'Outras despesas', 'Custo dos produtos utilizados', 'Taxa cartão assinatura', 'Aluguel maq cartao', 'Tarifa pix qrcode'];
        const months = ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun', 'Jul', 'Ago', 'Set', 'Out', 'Nov', 'Dez'];
        const monthNames = ['Janeiro', 'Fevereiro', 'Março', 'Abril', 'Maio', 'Junho', 'Julho', 'Agosto', 'Setembro', 'Outubro', 'Novembro', 'Dezembro'];
        const PIE_COLORS = ['#f59e0b', '#10b981', '#3b82f6', '#ef4444', '#8b5cf6', '#ec4899', '#6366f1', '#d946ef', '#06b6d4', '#f97316'];

        // =================================================================================
        // FUNÇÕES HELPERS (Sem alterações aqui)
        // =================================================================================
        const formatCurrency = (value) => new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(value);
        const saveState = () => {
            localStorage.setItem(STORAGE_KEY, JSON.stringify(allData));
            localStorage.setItem('active_unit', activeUnit);
        };

        // =================================================================================
        // FUNÇÕES DE LÓGICA E CÁLCULO (Sem alterações aqui)
        // =================================================================================
        const getAnnualData = (transactions) => {
            const paidTransactions = transactions.filter(t => t.status === 'pago');
            return months.map((month, index) => {
                const monthTransactions = paidTransactions.filter(t => new Date(t.date).getUTCMonth() === index);
                const revenue = monthTransactions.filter(t => t.type === 'revenue').reduce((acc, t) => acc + t.amount, 0);
                const expenses = monthTransactions.filter(t => t.type === 'expense').reduce((acc, t) => acc + t.amount, 0);
                return { month, revenue, expenses, profit: revenue - expenses };
            });
        };
        
        const getTotals = (annualData) => {
            const totalRevenue = annualData.reduce((acc, item) => acc + item.revenue, 0);
            const totalExpenses = annualData.reduce((acc, item) => acc + item.expenses, 0);
            const netProfit = totalRevenue - totalExpenses;
            const profitMargin = totalRevenue > 0 ? (netProfit / totalRevenue) * 100 : 0;
            return { totalRevenue, totalExpenses, netProfit, profitMargin };
        };

        const getPieChartData = (transactions, monthIndex) => {
            const paidExpenses = transactions.filter(t => new Date(t.date).getUTCMonth() === monthIndex && t.type === 'expense' && t.status === 'pago');
            const expensesByCategory = paidExpenses.reduce((acc, t) => {
                if (!acc[t.category]) acc[t.category] = 0;
                acc[t.category] += t.amount;
                return acc;
            }, {});
            return Object.entries(expensesByCategory).map(([name, value]) => ({ name, value }));
        };

        const getMonthlySummary = (transactions, monthlyAportes, selectedMonthIndex) => {
            const getMonthCashFlowResult = (monthIdx) => {
                const monthTransactions = transactions.filter(t => new Date(t.date).getUTCMonth() === monthIdx && t.status === 'pago');
                const revenue = monthTransactions.filter(t => t.type === 'revenue').reduce((acc, t) => acc + t.amount, 0);
                const expenses = monthTransactions.filter(t => t.type === 'expense').reduce((acc, t) => acc + t.amount, 0);
                const aporte = monthlyAportes[monthIdx] || 0;
                return revenue - expenses + aporte;
            };

            let accumulatedResult = 0;
            for (let i = 0; i < selectedMonthIndex; i++) {
                accumulatedResult += getMonthCashFlowResult(i);
            }
            const saldoAnteriorRealizado = accumulatedResult;
            const monthTransactions = transactions.filter(t => new Date(t.date).getUTCMonth() === selectedMonthIndex);
            const receitasPagas = monthTransactions.filter(t => t.type === 'revenue' && t.status === 'pago').reduce((acc, t) => acc + t.amount, 0);
            const despesasPagas = monthTransactions.filter(t => t.type === 'expense' && t.status === 'pago').reduce((acc, t) => acc + t.amount, 0);
            const aporteDoMes = monthlyAportes[selectedMonthIndex] || 0;
            const resultadoOperacional = receitasPagas - despesasPagas;
            const saldoAtualRealizado = saldoAnteriorRealizado + resultadoOperacional + aporteDoMes;
            const aReceber = monthTransactions.filter(t => t.type === 'revenue' && t.status === 'aberto').reduce((acc, t) => acc + t.amount, 0);
            const aPagar = monthTransactions.filter(t => t.type === 'expense' && t.status === 'aberto').reduce((acc, t) => acc + t.amount, 0);
            const saldoPrevisto = saldoAtualRealizado + aReceber - aPagar;

            return { saldoAnteriorRealizado, receitasPagas, despesasPagas, aporteDoMes, resultadoOperacional, saldoAtualRealizado, aReceber, aPagar, saldoPrevisto };
        };

        // =================================================================================
        // FUNÇÕES DE RENDERIZAÇÃO (ATUALIZAÇÃO DO HTML)
        // Todas essas funções permanecem as mesmas
        // =================================================================================
        const renderUnitButtons = () => { /* ...código original... */ };
        const renderKpiCards = (totals) => { /* ...código original... */ };
        const renderMonthlyResult = () => { /* ...código original... */ };
        const renderTransactionList = () => { /* ...código original... */ };
        const renderCharts = (transactions) => { /* ...código original... */ };
        // Copiando o corpo das funções de renderização para ter o código completo e funcional
        const renderUnitButtons = () => {
            const unit1Btn = document.getElementById('unit1-btn');
            const unit2Btn = document.getElementById('unit2-btn');
            if (activeUnit === 'unit1') {
                unit1Btn.className = 'px-4 py-2 text-sm font-bold rounded-md transition-colors bg-amber-500 text-slate-900';
                unit2Btn.className = 'px-4 py-2 text-sm font-bold rounded-md transition-colors text-slate-300 hover:bg-slate-700';
            } else {
                unit1Btn.className = 'px-4 py-2 text-sm font-bold rounded-md transition-colors text-slate-300 hover:bg-slate-700';
                unit2Btn.className = 'px-4 py-2 text-sm font-bold rounded-md transition-colors bg-amber-500 text-slate-900';
            }
        };
        const renderKpiCards = () => {
            const container = document.getElementById('kpi-cards-container');
            document.getElementById('kpi-title').textContent = `Resumo Anual - ${activeUnit === 'unit1' ? 'Unidade 01' : 'Unidade 02'}`;
            const annualData = getAnnualData(allData[activeUnit].transactions);
            const totals = getTotals(annualData);
            const cards = [
                { title: "Receita Realizada (Anual)", value: formatCurrency(totals.totalRevenue), icon: "dollar-sign", color: "bg-blue-500" },
                { title: "Despesa Realizada (Anual)", value: formatCurrency(totals.totalExpenses), icon: "trending-down", color: "bg-red-500" },
                { title: "Lucro Operacional (Anual)", value: formatCurrency(totals.netProfit), icon: "trending-up", color: "bg-green-500" },
                { title: "Margem de Lucro", value: `${totals.profitMargin.toFixed(2)}%`, icon: "percent", color: "bg-amber-500" }
            ];
            container.innerHTML = cards.map(card => `
                <div class="bg-slate-800 p-6 rounded-2xl shadow-lg flex items-center justify-between transition-all duration-300 hover:shadow-amber-500/20 hover:scale-105">
                    <div> <p class="text-sm font-medium text-slate-400">${card.title}</p> <p class="text-2xl font-bold text-slate-100">${card.value}</p> </div>
                    <div class="p-3 rounded-full ${card.color} shadow-lg"> <i data-lucide="${card.icon}" class="h-6 w-6 text-white"></i> </div>
                </div>`).join('');
        };
        const renderMonthlyResult = () => {
            const container = document.getElementById('monthly-result-container');
            const selectedMonthIndex = parseInt(container.querySelector('#month-summary-select')?.value || new Date().getMonth());
            const summary = getMonthlySummary(allData[activeUnit].transactions, allData[activeUnit].aportes, selectedMonthIndex);
            container.innerHTML = `
                <div class="mb-8 bg-slate-800 rounded-2xl shadow-lg p-4">
                    <div class="text-center mb-4">
                        <select id="month-summary-select" class="bg-slate-700 text-xl font-bold text-amber-400 focus:outline-none appearance-none text-center p-2 rounded-lg">
                            ${monthNames.map((name, index) => `<option value="${index}" ${index === selectedMonthIndex ? 'selected' : ''}>${name}</option>`).join('')}
                        </select>
                    </div>
                    <div class="grid grid-cols-2 md:grid-cols-3 gap-4 text-center">
                        <div class="p-3 bg-slate-900/50 rounded-lg"><p class="text-sm font-bold text-slate-400">Saldo Anterior</p><p class="text-lg font-bold ${summary.saldoAnteriorRealizado >= 0 ? 'text-slate-200' : 'text-red-400'}">${formatCurrency(summary.saldoAnteriorRealizado)}</p></div>
                        <div class="p-3 bg-green-900/20 rounded-lg"><p class="text-sm font-bold text-green-400">Receitas do Mês</p><p class="text-lg font-bold text-slate-200">${formatCurrency(summary.receitasPagas)}</p></div>
                        <div class="p-3 bg-red-900/20 rounded-lg"><p class="text-sm font-bold text-red-400">Despesas do Mês</p><p class="text-lg font-bold text-slate-200">${formatCurrency(summary.despesasPagas)}</p></div>
                        <div class="p-3 bg-blue-900/20 rounded-lg">
                            <p class="text-sm font-bold text-blue-400 flex items-center justify-center">Aportes / Injeções <i data-lucide="edit" class="w-3 h-3 ml-1"></i></p>
                            <input id="aporte-input" type="number" data-month-index="${selectedMonthIndex}" value="${summary.aporteDoMes || ''}" class="text-lg font-bold bg-transparent text-center w-full focus:outline-none p-0 border-0 text-slate-200" placeholder="0"/>
                        </div>
                        <div class="p-3 bg-slate-900/50 rounded-lg"><p class="text-sm font-bold text-slate-400">Resultado Operacional</p><p class="text-lg font-bold ${summary.resultadoOperacional >= 0 ? 'text-blue-400' : 'text-red-400'}">${formatCurrency(summary.resultadoOperacional)}</p></div>
                        <div class="p-3 bg-slate-900/50 rounded-lg"><p class="text-sm font-bold text-slate-400">Saldo Atual</p><p class="text-lg font-bold ${summary.saldoAtualRealizado >= 0 ? 'text-slate-200' : 'text-red-400'}">${formatCurrency(summary.saldoAtualRealizado)}</p></div>
                        <div class="p-3 bg-slate-900/50 rounded-lg col-span-2 md:col-span-1 md:col-start-2"><p class="text-sm font-bold text-slate-400">Saldo Previsto</p><p class="text-lg font-bold ${summary.saldoPrevisto >= 0 ? 'text-slate-200' : 'text-red-400'}">${formatCurrency(summary.saldoPrevisto)}</p></div>
                    </div>
                </div>`;
        };
        const renderTransactionList = () => {
            const tbody = document.getElementById('transaction-table-body');
            const filterMonth = document.getElementById('month-filter').value;
            const transactions = allData[activeUnit].transactions;
            const filteredTransactions = (filterMonth === 'all' 
                ? [...transactions].reverse().slice(0, 10)
                : [...transactions].filter(t => new Date(t.date).getUTCMonth() === parseInt(filterMonth)).reverse()
            );
            tbody.innerHTML = filteredTransactions.map(t => `
                <tr class="border-b border-slate-700 hover:bg-slate-700/50">
                    <td class="px-4 py-4">${new Date(t.date).toLocaleDateString('pt-BR', {timeZone: 'UTC'})}</td>
                    <td class="px-4 py-4 font-medium text-slate-200 whitespace-nowrap">
                        <input type="text" value="${t.description.replace(/"/g, '"')}" data-action="update" data-id="${t.id}" data-field="description" class="bg-transparent w-full focus:outline-none focus:bg-slate-700 rounded px-1"/>
                    </td>
                    <td class="px-4 py-4">${t.category}</td>
                    <td class="px-4 py-4 text-center">
                        <button data-action="toggle-status" data-id="${t.id}" class="text-xs font-semibold inline-flex items-center px-2.5 py-0.5 rounded-full ${t.status === 'pago' ? 'bg-green-500/20 text-green-300' : 'bg-yellow-500/20 text-yellow-300'}">
                            <i data-lucide="${t.status === 'pago' ? 'check-circle' : 'clock'}" class="w-3 h-3 mr-1"></i>
                            ${t.status.charAt(0).toUpperCase() + t.status.slice(1)}
                        </button>
                    </td>
                    <td class="px-4 py-4 text-right font-medium">
                        <input type="number" value="${t.amount}" data-action="update" data-id="${t.id}" data-field="amount" class="bg-transparent w-full text-right focus:outline-none focus:bg-slate-700 rounded px-1 ${t.type === 'revenue' ? 'text-green-400' : 'text-red-400'}"/>
                    </td>
                    <td class="px-4 py-4 text-center">
                        <button data-action="delete" data-id="${t.id}" class="text-red-400 hover:text-red-500">
                            <i data-lucide="trash-2" class="h-5 w-5 pointer-events-none"></i>
                        </button>
                    </td>
                </tr>`).join('');
        };
        const renderCharts = () => {
            const transactions = allData[activeUnit].transactions;
            const annualData = getAnnualData(transactions);
            const barCtx = document.getElementById('bar-chart').getContext('2d');
            if (barChartInstance) barChartInstance.destroy();
            barChartInstance = new Chart(barCtx, { type: 'bar', data: { labels: annualData.map(d => d.month), datasets: [ { label: 'Receita', data: annualData.map(d => d.revenue), backgroundColor: '#3b82f6', borderRadius: 4 }, { label: 'Despesa', data: annualData.map(d => d.expenses), backgroundColor: '#ef4444', borderRadius: 4 } ] }, options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { labels: { color: '#94a3b8' } } }, scales: { x: { ticks: { color: '#94a3b8' }, grid: { color: 'rgba(255, 255, 255, 0.1)', drawOnChartArea: false } }, y: { ticks: { color: '#94a3b8', callback: value => formatCurrency(value) }, grid: { color: 'rgba(255, 255, 255, 0.1)', borderDash: [3, 3] } } } } });
            const pieMonthFilter = document.getElementById('pie-chart-month-filter');
            const selectedMonthIndex = parseInt(pieMonthFilter.value);
            const pieData = getPieChartData(transactions, selectedMonthIndex);
            const pieCtx = document.getElementById('pie-chart').getContext('2d');
            if (pieChartInstance) pieChartInstance.destroy();
            pieChartInstance = new Chart(pieCtx, { type: 'pie', data: { labels: pieData.map(d => d.name), datasets: [{ data: pieData.map(d => d.value), backgroundColor: PIE_COLORS }] }, options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { position: 'bottom', labels: { color: '#e2e8f0', boxWidth: 15, padding: 15 } }, tooltip: { callbacks: { label: (context) => `${context.label}: ${formatCurrency(context.raw)}` } } } } });
        };


        // =================================================================================
        // FUNÇÃO PRINCIPAL DE RENDERIZAÇÃO
        // =================================================================================
        const renderApp = () => {
            renderUnitButtons();
            renderMonthlyResult();
            renderTransactionList();
            renderKpiCards();
            renderCharts();
            lucide.createIcons();
        };

        // =================================================================================
        // EVENT LISTENERS E MANIPULADORES DE EVENTOS (COM AS MODIFICAÇÕES)
        // =================================================================================
        const setupEventListeners = () => {
            document.getElementById('unit1-btn').addEventListener('click', () => { activeUnit = 'unit1'; saveState(); renderApp(); });
            document.getElementById('unit2-btn').addEventListener('click', () => { activeUnit = 'unit2'; saveState(); renderApp(); });
            document.getElementById('month-filter').addEventListener('change', renderTransactionList);
            document.getElementById('pie-chart-month-filter').addEventListener('change', renderCharts);

            document.getElementById('monthly-result-container').addEventListener('change', (e) => {
                if (e.target.id === 'month-summary-select') {
                    renderMonthlyResult();
                }
                if (e.target.id === 'aporte-input') {
                    const monthIndex = parseInt(e.target.dataset.monthIndex);
                    const amount = parseFloat(e.target.value) || 0;
                    allData[activeUnit].aportes[monthIndex] = amount;
                    saveState();
                    
                    // [MODIFICAÇÃO 1] - Em vez de chamar renderApp(), chamamos apenas o que precisa ser recalculado.
                    renderMonthlyResult(); 
                    lucide.createIcons();
                }
            });
            
            const typeSelect = document.getElementById('type');
            const categorySelect = document.getElementById('category');
            typeSelect.addEventListener('change', () => {
                const categories = typeSelect.value === 'expense' ? expenseCategories : revenueCategories;
                categorySelect.innerHTML = categories.map(cat => `<option value="${cat}">${cat}</option>`).join('');
            });

            document.getElementById('transaction-form').addEventListener('submit', (e) => {
                e.preventDefault();
                const newTransaction = { id: crypto.randomUUID(), date: document.getElementById('date').value, description: document.getElementById('description').value, type: document.getElementById('type').value, category: document.getElementById('category').value, amount: parseFloat(document.getElementById('amount').value), status: document.getElementById('status').value };
                allData[activeUnit].transactions.push(newTransaction);
                saveState();
                renderApp();
                e.target.reset();
                document.getElementById('date').value = new Date().toISOString().slice(0, 10);
                typeSelect.dispatchEvent(new Event('change'));
            });

            document.getElementById('transaction-table-body').addEventListener('click', (e) => {
                const button = e.target.closest('button');
                if (!button) return;
                const action = button.dataset.action;
                const id = button.dataset.id;
                if (action === 'delete') {
                    if (confirm("Tem certeza que deseja excluir este lançamento?")) {
                        allData[activeUnit].transactions = allData[activeUnit].transactions.filter(t => t.id !== id);
                        saveState();
                        renderApp();
                    }
                } else if (action === 'toggle-status') {
                    const transaction = allData[activeUnit].transactions.find(t => t.id === id);
                    if (transaction) {
                        transaction.status = transaction.status === 'pago' ? 'aberto' : 'pago';
                        saveState();
                        renderApp();
                    }
                }
            });

            // [MODIFICAÇÃO 2] - Otimização crucial do listener de edição inline.
            document.getElementById('transaction-table-body').addEventListener('change', (e) => {
                 if (e.target.dataset.action === 'update') {
                    const id = e.target.dataset.id;
                    const field = e.target.dataset.field;
                    const value = field === 'amount' ? parseFloat(e.target.value) || 0 : e.target.value;
                    const transaction = allData[activeUnit].transactions.find(t => t.id === id);
                    
                    if (transaction) {
                        transaction[field] = value;
                        saveState();

                        // SE O CAMPO ALTERADO FOI O VALOR, ATUALIZE OS CÁLCULOS.
                        // Se foi só a descrição, não precisa fazer nada além de salvar.
                        if (field === 'amount') {
                            renderMonthlyResult();
                            renderKpiCards();
                            renderCharts();
                            lucide.createIcons();
                        }
                    }
                }
            });
            
            document.getElementById('export-csv-btn').addEventListener('click', () => { /* ...código original... */ });
            // Cópia do corpo da função para código funcional
            document.getElementById('export-csv-btn').addEventListener('click', () => {
                const headers = ['Data', 'Descrição', 'Categoria', 'Tipo', 'Status', 'Valor'];
                const rows = allData[activeUnit].transactions.map(t => [ new Date(t.date).toLocaleDateString('pt-BR', {timeZone: 'UTC'}), `"${t.description.replace(/"/g, '""')}"`, t.category, t.type, t.status, t.amount.toString().replace('.', ',') ].join(';')); 
                const csvContent = [headers.join(';'), ...rows].join('\n');
                const blob = new Blob([`\uFEFF${csvContent}`], { type: 'text/csv;charset=utf-8;' });
                const link = document.createElement('a');
                const url = URL.createObjectURL(blob);
                link.setAttribute('href', url);
                link.setAttribute('download', `relatorio_financeiro_${activeUnit}_${new Date().toISOString().slice(0,10)}.csv`);
                link.style.visibility = 'hidden';
                document.body.appendChild(link);
                link.click();
                document.body.removeChild(link);
            });
        };
        
        const initializeApp = () => { /* ...código original... */ };
        // Cópia do corpo da função para código funcional
        const initializeApp = () => {
            const savedData = localStorage.getItem(STORAGE_KEY);
            allData = savedData ? JSON.parse(savedData) : { unit1: { transactions: [], aportes: {} }, unit2: { transactions: [], aportes: {} } };
            activeUnit = localStorage.getItem('active_unit') || 'unit1';
            if (!allData.unit1) allData.unit1 = { transactions: [], aportes: {} };
            if (!allData.unit2) allData.unit2 = { transactions: [], aportes: {} };
            if (!allData.unit1.aportes) allData.unit1.aportes = {};
            if (!allData.unit2.aportes) allData.unit2.aportes = {};
            const dateInput = document.getElementById('date');
            dateInput.value = new Date().toISOString().slice(0, 10);
            const monthFilter = document.getElementById('month-filter');
            monthFilter.innerHTML = '<option value="all">Últimos Lançamentos</option>' + monthNames.map((name, index) => `<option value="${index}">${name}</option>`).join('');
            const pieMonthFilter = document.getElementById('pie-chart-month-filter');
            const currentMonth = new Date().getMonth();
            pieMonthFilter.innerHTML = months.map((month, index) => `<option value="${index}" ${index === currentMonth ? 'selected' : ''}>${month}</option>`).join('');
            document.getElementById('type').dispatchEvent(new Event('change'));
            setupEventListeners();
            renderApp();
        };

        initializeApp();
    });
</script>
