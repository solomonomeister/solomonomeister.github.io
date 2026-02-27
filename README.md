<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BurnTrack v10</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&amp;display=swap');
        
        :root {
            --accent: 99 102 241;
        }
        
        body {
            font-family: 'Inter', system-ui, sans-serif;
        }
        
        .tail-container {
            font-family: 'Inter', system-ui, sans-serif;
        }
        
        details summary {
            list-style: none;
        }
        
        details summary::-webkit-details-marker {
            display: none;
        }
        
        .chevron {
            transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
        }
        
        details[open] .chevron {
            transform: rotate(180deg);
        }
        
        .money-label {
            font-feature-settings: "tnum";
        }
        
        .card {
            transition: transform 0.2s cubic-bezier(0.4, 0, 0.2, 1);
        }
        
        .card:hover {
            transform: translateY(-2px);
        }
        
        input[type="range"] {
            accent-color: rgb(99 102 241);
        }
        
        .table-row {
            transition: background-color 0.1s ease;
        }
        
        .table-row:hover {
            background-color: rgb(39 39 42);
        }
    </style>
</head>
<body class="tail-container bg-[#0a0a0a] text-white min-h-screen pb-12">
    <div class="max-w-7xl mx-auto px-8">
        <!-- HEADER -->
        <header class="flex items-center justify-between py-8 border-b border-zinc-800">
            <div class="flex items-center gap-x-4">
                <div class="w-9 h-9 bg-indigo-600 rounded-2xl flex items-center justify-center text-2xl font-bold shadow-inner">B</div>
                <div>
                    <h1 class="text-3xl font-semibold tracking-tighter">BurnTrack</h1>
                    <p class="text-xs tracking-[2px] text-zinc-500 font-medium">MARCH 2026 • MELBOURNE • FULL LOAN MODEL</p>
                </div>
            </div>
            <div class="flex items-center gap-x-3">
                <button onclick="resetApp()" 
                        class="flex items-center gap-x-2 px-6 py-3 rounded-3xl bg-zinc-900 hover:bg-zinc-800 border border-zinc-700 text-sm font-medium transition-all active:scale-95">
                    <i class="fa-solid fa-rotate"></i>
                    <span>Reset</span>
                </button>
                <button onclick="exportCSV()" 
                        class="flex items-center gap-x-2 px-6 py-3 rounded-3xl bg-indigo-600 hover:bg-indigo-500 text-sm font-medium transition-all active:scale-95">
                    <i class="fa-solid fa-download"></i>
                    <span>Export CSV</span>
                </button>
            </div>
        </header>

        <!-- SUMMARY ROW -->
        <div id="summary" class="grid grid-cols-5 gap-6 mt-10">
            <!-- JS injected -->
        </div>

        <!-- PROJECT INPUTS -->
        <div class="mt-16">
            <div class="flex items-center gap-x-3 mb-8">
                <i class="fa-solid fa-sliders text-indigo-400 text-2xl"></i>
                <h2 class="text-2xl font-semibold tracking-tight">PROJECT INPUTS</h2>
            </div>

            <!-- COLLAPSIBLES -->
            <div class="space-y-4" id="inputs-container">
                <!-- Populated by JS -->
            </div>
        </div>

        <!-- VISUAL AREA -->
        <div class="grid grid-cols-5 gap-6 mt-16">
            <!-- CHART -->
            <div class="col-span-3 bg-zinc-900 border border-zinc-800 rounded-3xl p-8">
                <div class="flex justify-between items-baseline mb-6">
                    <h3 class="font-semibold text-lg">Projected Cash on Hand</h3>
                    <div class="text-xs text-zinc-500 font-mono">MAR 26 — AUG 28</div>
                </div>
                <canvas id="cashChart" class="w-full" height="340"></canvas>
            </div>

            <!-- PHASE PROGRESS -->
            <div class="col-span-2 bg-zinc-900 border border-zinc-800 rounded-3xl p-8">
                <h3 class="font-semibold text-lg mb-6">Phase Progress</h3>
                <div id="phase-progress" class="space-y-7">
                    <!-- JS injected -->
                </div>
            </div>
        </div>

        <!-- TIMELINE TABLE -->
        <div class="mt-16 bg-zinc-900 border border-zinc-800 rounded-3xl overflow-hidden">
            <div class="px-8 py-6 border-b border-zinc-800 flex items-center justify-between">
                <h3 class="font-semibold text-lg">30-Month Timeline</h3>
                <div class="text-xs uppercase tracking-widest text-zinc-500 font-medium">Cash on hand shown at month end</div>
            </div>
            <div class="overflow-auto max-h-[520px]">
                <table class="w-full text-sm" id="timeline-table">
                    <thead class="sticky top-0 bg-zinc-900 border-b border-zinc-800">
                        <tr class="text-left text-zinc-400">
                            <th class="px-8 py-5 font-medium w-32">MONTH</th>
                            <th class="px-8 py-5 font-medium">MONTHLY OUTFLOW</th>
                            <th class="px-8 py-5 font-medium">CASH ON HAND</th>
                        </tr>
                    </thead>
                    <tbody id="timeline-body" class="text-zinc-300 font-mono divide-y divide-zinc-800">
                        <!-- JS injected -->
                    </tbody>
                </table>
            </div>
        </div>
    </div>

    <script>
        // Tailwind script run
        function initTailwind() {
            // Already loaded via CDN
        }

        // Money formatter
        function formatMoney(val) {
            if (val >= 1000000) {
                const m = (val / 1000000).toFixed(2);
                return '$' + m + 'M';
            } else {
                const k = Math.round(val / 1000);
                return '$' + k + 'k';
            }
        }

        // Stamp duty VIC 2026 general rates
        function calculateStampDuty(price) {
            if (price <= 25000) return price * 0.014;
            if (price <= 130000) return 350 + (price - 25000) * 0.024;
            if (price <= 960000) return 2870 + (price - 130000) * 0.06;
            return 52070 + (price - 960000) * 0.055;
        }

        // Default values
        let inputs = {
            purchasePrice: 1050000,
            settlementDays: 60,
            conveyancing: 2500,
            prePurchaseCash: 500000,
            interestRate: 6.0,
            surveyor: 2500,
            structuralEngineer: 2500,
            insurance: 1500,
            dilapidationReport: 1500,
            soilTest: 1000,
            demolitionWorks: 30000,
            makeGoodWorks: 3750,
            architecturePlans: 22500,
            engineeringPlans: 12500,
            slab: 100000,
            frame: 125000,
            roof: 75000,
            internal: 150000,
            finishing: 50000,
            rentPerWeek: 500,
            livingExpenses: 3750,
            contingency: 15,
            djTint: 8000,
            alexTint: 6000,
            djRodeo: 4000
        };

        let cashChartInstance = null;

        function createSlider(id, min, max, step, defaultVal, label, unit = '', isPercent = false) {
            const div = document.createElement('div');
            div.className = 'space-y-3';
            div.innerHTML = `
                <div class="flex justify-between text-sm">
                    <span class="text-zinc-400">${label}</span>
                    <span id="${id}Val" class="font-semibold font-mono text-indigo-400 money-label">${isPercent ? defaultVal + '%' : formatMoney(defaultVal)}</span>
                </div>
                <input type="range" id="${id}" min="${min}" max="${max}" step="${step}" value="${defaultVal}" 
                       class="w-full accent-indigo-500">
            `;
            return div;
        }

        function renderInputs() {
            const container = document.getElementById('inputs-container');
            container.innerHTML = '';

            // Purchase Phase
            let details = createDetails('Purchase Phase', 'fa-house', `
                ${createSlider('purchasePrice', 900000, 1200000, 5000, 1050000, 'Purchase Price').outerHTML}
                ${createSlider('settlementDays', 30, 90, 5, 60, 'Settlement Days', ' days').outerHTML}
                ${createSlider('conveyancing', 2000, 3000, 100, 2500, 'Conveyancing').outerHTML}
                ${createSlider('prePurchaseCash', 450000, 550000, 5000, 500000, 'Pre-purchase Cash').outerHTML}
            `);
            container.appendChild(details);

            // Financing
            details = createDetails('Financing', 'fa-hand-holding-dollar', `
                ${createSlider('interestRate', 5, 7, 0.1, 6.0, 'Interest Rate %', '%', true).outerHTML}
            `);
            container.appendChild(details);

            // Demolition
            details = createDetails('Demolition', 'fa-wrecking-ball', `
                ${createSlider('surveyor', 2000, 3000, 100, 2500, 'Surveyor').outerHTML}
                ${createSlider('structuralEngineer', 2000, 3000, 100, 2500, 'Structural Engineer').outerHTML}
                ${createSlider('insurance', 1000, 2000, 100, 1500, 'Insurance').outerHTML}
                ${createSlider('dilapidationReport', 1000, 2000, 100, 1500, 'Dilapidation Report').outerHTML}
                <div class="flex justify-between text-sm pt-2">
                    <span class="text-zinc-400">Soil Test</span>
                    <span class="font-semibold font-mono text-indigo-400">$1k</span>
                </div>
                ${createSlider('demolitionWorks', 25000, 35000, 500, 30000, 'Demolition Works').outerHTML}
                ${createSlider('makeGoodWorks', 2500, 5000, 100, 3750, 'Make Good Works').outerHTML}
            `);
            container.appendChild(details);

            // Pre-Construction Plans
            details = createDetails('Pre-Construction Plans', 'fa-drafting-compass', `
                ${createSlider('architecturePlans', 15000, 30000, 500, 22500, 'Architecture Plans').outerHTML}
                ${createSlider('engineeringPlans', 10000, 15000, 500, 12500, 'Engineering Plans').outerHTML}
            `);
            container.appendChild(details);

            // New Build Stages
            details = createDetails('New Build Stages', 'fa-trowel-bricks', `
                ${createSlider('slab', 70000, 156000, 1000, 100000, 'Slab / Foundation').outerHTML}
                ${createSlider('frame', 70000, 195000, 1000, 125000, 'Frame / Structure').outerHTML}
                ${createSlider('roof', 42000, 117000, 1000, 75000, 'Roof / Lock-up').outerHTML}
                ${createSlider('internal', 84000, 234000, 1000, 150000, 'Internal Fit-out').outerHTML}
                ${createSlider('finishing', 28000, 78000, 1000, 50000, 'Finishing / Handover').outerHTML}
            `);
            container.appendChild(details);

            // Holding & Living Costs
            details = createDetails('Holding & Living Costs', 'fa-key', `
                ${createSlider('rentPerWeek', 0, 1000, 10, 500, 'Rent per week', ' /wk').outerHTML}
                ${createSlider('livingExpenses', 2500, 5000, 100, 3750, 'Living Expenses monthly').outerHTML}
                ${createSlider('contingency', 10, 20, 1, 15, 'Contingency % (demo + build)', '%', true).outerHTML}
            `);
            container.appendChild(details);

            // Monthly Income
            details = createDetails('Monthly Income', 'fa-sack-dollar', `
                ${createSlider('djTint', 6000, 10000, 100, 8000, 'DJ Tint Salary').outerHTML}
                ${createSlider('alexTint', 5000, 7000, 100, 6000, 'Alex Tint Salary').outerHTML}
                ${createSlider('djRodeo', 2000, 6000, 100, 4000, 'DJ Rodeo Salary').outerHTML}
            `);
            container.appendChild(details);

            // Attach listeners
            setTimeout(() => {
                const allSliders = container.querySelectorAll('input[type="range"]');
                allSliders.forEach(slider => {
                    slider.addEventListener('input', () => {
                        updateSliderValue(slider);
                        debounceCalculate();
                    });
                });
            }, 100);
        }

        function createDetails(title, icon, contentHTML) {
            const detailsEl = document.createElement('details');
            detailsEl.className = 'group bg-zinc-900 border border-zinc-800 rounded-3xl';
            detailsEl.innerHTML = `
                <summary class="px-8 py-6 flex items-center justify-between cursor-pointer hover:bg-zinc-800/50 rounded-3xl">
                    <div class="flex items-center gap-x-4">
                        <i class="fa-solid ${icon} text-indigo-400 w-5"></i>
                        <span class="font-medium text-lg">${title}</span>
                    </div>
                    <i class="fa-solid fa-chevron-down chevron text-zinc-400"></i>
                </summary>
                <div class="px-8 pb-8 pt-2 space-y-8 border-t border-zinc-800">
                    ${contentHTML}
                </div>
            `;
            return detailsEl;
        }

        function updateSliderValue(slider) {
            const valSpan = document.getElementById(slider.id + 'Val');
            if (!valSpan) return;
            const val = parseFloat(slider.value);
            if (slider.id === 'interestRate' || slider.id === 'contingency') {
                valSpan.textContent = val + (slider.id === 'interestRate' ? '.0' : '') + '%';
            } else if (slider.id === 'settlementDays') {
                valSpan.textContent = val + ' days';
            } else if (slider.id === 'rentPerWeek') {
                valSpan.textContent = '$' + val + '/wk';
            } else {
                valSpan.textContent = formatMoney(val);
            }
        }

        let debounceTimer = null;
        function debounceCalculate() {
            clearTimeout(debounceTimer);
            debounceTimer = setTimeout(() => {
                calculateAndRenderAll();
            }, 420);
        }

        function getCurrentInputs() {
            return {
                purchasePrice: parseFloat(document.getElementById('purchasePrice').value),
                settlementDays: parseFloat(document.getElementById('settlementDays').value),
                conveyancing: parseFloat(document.getElementById('conveyancing').value),
                prePurchaseCash: parseFloat(document.getElementById('prePurchaseCash').value),
                interestRate: parseFloat(document.getElementById('interestRate').value),
                surveyor: parseFloat(document.getElementById('surveyor').value),
                structuralEngineer: parseFloat(document.getElementById('structuralEngineer').value),
                insurance: parseFloat(document.getElementById('insurance').value),
                dilapidationReport: parseFloat(document.getElementById('dilapidationReport').value),
                soilTest: 1000,
                demolitionWorks: parseFloat(document.getElementById('demolitionWorks').value),
                makeGoodWorks: parseFloat(document.getElementById('makeGoodWorks').value),
                architecturePlans: parseFloat(document.getElementById('architecturePlans').value),
                engineeringPlans: parseFloat(document.getElementById('engineeringPlans').value),
                slab: parseFloat(document.getElementById('slab').value),
                frame: parseFloat(document.getElementById('frame').value),
                roof: parseFloat(document.getElementById('roof').value),
                internal: parseFloat(document.getElementById('internal').value),
                finishing: parseFloat(document.getElementById('finishing').value),
                rentPerWeek: parseFloat(document.getElementById('rentPerWeek').value),
                livingExpenses: parseFloat(document.getElementById('livingExpenses').value),
                contingency: parseFloat(document.getElementById('contingency').value),
                djTint: parseFloat(document.getElementById('djTint').value),
                alexTint: parseFloat(document.getElementById('alexTint').value),
                djRodeo: parseFloat(document.getElementById('djRodeo').value)
            };
        }

        function calculateAndRenderAll() {
            const inp = getCurrentInputs();
            
            // Calculations
            const stampDuty = calculateStampDuty(inp.purchasePrice);
            const purchaseLoan = Math.round(inp.purchasePrice * 0.8);
            const purchaseDeposit = Math.round(inp.purchasePrice * 0.2 + stampDuty + inp.conveyancing);
            
            const demoRaw = inp.surveyor + inp.structuralEngineer + inp.insurance + inp.dilapidationReport + inp.soilTest + inp.demolitionWorks + inp.makeGoodWorks;
            const totalDemo = Math.round(demoRaw * (1 + inp.contingency / 100));
            
            const preRaw = inp.architecturePlans + inp.engineeringPlans;
            
            const buildRaw = inp.slab + inp.frame + inp.roof + inp.internal + inp.finishing;
            const totalBuild = Math.round(buildRaw * (1 + inp.contingency / 100));
            
            const totalProjectCost = inp.purchasePrice + stampDuty + inp.conveyancing + totalDemo + preRaw + totalBuild;
            
            const monthlyRent = Math.round(inp.rentPerWeek * 52 / 12);
            const monthlyHolding = monthlyRent + inp.livingExpenses;
            const monthlyIncome = inp.djTint + inp.alexTint + inp.djRodeo;
            
            const constructionLoan = Math.round(totalBuild * 0.8);
            const totalLoan = purchaseLoan + constructionLoan;
            const monthlyRate = inp.interestRate / 1200;
            
            // P+I from month 26
            let monthlyPI = 0;
            if (totalLoan > 0) {
                const r = monthlyRate;
                const n = 360;
                monthlyPI = Math.round(totalLoan * r * Math.pow(1 + r, n) / (Math.pow(1 + r, n) - 1));
            }
            
            const settlementMonth = Math.ceil(inp.settlementDays / 30);
            
            // Simulate 30 months
            let cash = inp.prePurchaseCash;
            const cashHistory = [];
            const outflowHistory = [];
            const monthLabels = [];
            
            let currentDate = new Date(2026, 2, 1); // March 2026
            
            let minCash = cash;
            let runwayMonth = 30;
            
            // Phase outflows
            const phaseOut = new Array(31).fill(0);
            if (settlementMonth <= 30) phaseOut[settlementMonth] += purchaseDeposit;
            
            // Demo spread 3 months after settlement
            let demoM = settlementMonth + 1;
            for (let i = 0; i < 3; i++) {
                if (demoM + i <= 30) phaseOut[demoM + i] += Math.round(totalDemo / 3);
            }
            
            // Pre-con at month 6
            if (6 <= 30) phaseOut[6] += preRaw;
            
            // Build deposit month 7
            if (7 <= 30) phaseOut[7] += Math.round(totalBuild * 0.2);
            
            for (let m = 1; m <= 30; m++) {
                let outflow = monthlyHolding;
                
                // Interest
                let interestThisMonth = 0;
                if (m >= settlementMonth) interestThisMonth += purchaseLoan * monthlyRate;
                if (m >= 7) interestThisMonth += constructionLoan * monthlyRate;
                
                if (m >= 26) {
                    outflow += monthlyPI;
                } else {
                    outflow += interestThisMonth;
                }
                
                outflow += phaseOut[m];
                
                const net = monthlyIncome - outflow;
                cash = Math.round(cash + net);
                
                cashHistory.push(cash);
                outflowHistory.push(Math.round(outflow));
                
                if (cash < minCash) minCash = cash;
                if (cash < 0 && runwayMonth === 30) runwayMonth = m;
                
                // Month label
                const label = currentDate.toLocaleString('en-US', { month: 'short' }).toUpperCase() + ' ' + currentDate.getFullYear().toString().slice(2);
                monthLabels.push(label);
                currentDate.setMonth(currentDate.getMonth() + 1);
            }
            
            // Summary cards
            renderSummary(inp.prePurchaseCash, monthlyIncome, minCash, runwayMonth, totalProjectCost);
            
            // Chart
            renderChart(monthLabels, cashHistory);
            
            // Phase progress
            renderPhaseProgress([
                { name: 'Purchase', cost: inp.purchasePrice + stampDuty + inp.conveyancing, color: 'indigo' },
                { name: 'Demolition', cost: totalDemo, color: 'violet' },
                { name: 'Pre-Construction', cost: preRaw, color: 'sky' },
                { name: 'New Build', cost: totalBuild, color: 'emerald' }
            ], totalProjectCost);
            
            // Timeline table
            renderTimeline(monthLabels, outflowHistory, cashHistory);
        }

        function renderSummary(startCash, monthlyInc, lowest, runway, totalProj) {
            const html = `
                <div class="bg-zinc-900 border border-zinc-800 rounded-3xl p-6 card">
                    <div class="text-xs text-zinc-500 tracking-widest">STARTING CASH</div>
                    <div class="text-4xl font-semibold mt-3 font-mono">${formatMoney(startCash)}</div>
                </div>
                <div class="bg-zinc-900 border border-zinc-800 rounded-3xl p-6 card">
                    <div class="text-xs text-zinc-500 tracking-widest">MONTHLY INCOME</div>
                    <div class="text-4xl font-semibold mt-3 font-mono">${formatMoney(monthlyInc)}</div>
                </div>
                <div class="bg-zinc-900 border border-zinc-800 rounded-3xl p-6 card ${lowest < 0 ? 'ring-1 ring-red-500/30' : ''}">
                    <div class="text-xs text-zinc-500 tracking-widest">LOWEST CASH</div>
                    <div class="text-4xl font-semibold mt-3 font-mono ${lowest < 0 ? 'text-red-400' : ''}">${formatMoney(lowest)}</div>
                </div>
                <div class="bg-zinc-900 border border-zinc-800 rounded-3xl p-6 card">
                    <div class="text-xs text-zinc-500 tracking-widest">RUNWAY</div>
                    <div class="text-4xl font-semibold mt-3 font-mono">${runway === 30 ? '≥30' : runway} <span class="text-base font-normal text-zinc-400">mo</span></div>
                </div>
                <div class="bg-zinc-900 border border-zinc-800 rounded-3xl p-6 card">
                    <div class="text-xs text-zinc-500 tracking-widest">TOTAL PROJECT</div>
                    <div class="text-4xl font-semibold mt-3 font-mono">${formatMoney(totalProj)}</div>
                </div>
            `;
            document.getElementById('summary').innerHTML = html;
        }

        function renderChart(labels, data) {
            const ctx = document.getElementById('cashChart');
            if (cashChartInstance) cashChartInstance.destroy();
            
            cashChartInstance = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: labels,
                    datasets: [{
                        label: 'Cash on Hand',
                        data: data,
                        borderColor: '#6366f1',
                        borderWidth: 3,
                        tension: 0.3,
                        pointRadius: 0,
                        pointHoverRadius: 5,
                        pointBackgroundColor: '#fff',
                        pointBorderWidth: 2
                    }]
                },
                options: {
                    plugins: {
                        legend: { display: false },
                        tooltip: {
                            backgroundColor: '#18181b',
                            titleColor: '#a1a1aa',
                            bodyColor: '#fff',
                            displayColors: false,
                            callbacks: {
                                label: (ctx) => formatMoney(ctx.raw)
                            }
                        }
                    },
                    scales: {
                        y: {
                            grid: { color: '#27272a' },
                            ticks: {
                                color: '#71717a',
                                callback: (v) => formatMoney(v)
                            }
                        },
                        x: {
                            grid: { color: '#27272a' },
                            ticks: { color: '#71717a', maxRotation: 45, minRotation: 45 }
                        }
                    },
                    interaction: {
                        intersect: false,
                        mode: 'index'
                    }
                }
            });
        }

        function renderPhaseProgress(phases, total) {
            const container = document.getElementById('phase-progress');
            container.innerHTML = '';
            
            phases.forEach(phase => {
                const percent = Math.round((phase.cost / total) * 100);
                const div = document.createElement('div');
                div.innerHTML = `
                    <div class="flex justify-between text-sm mb-1.5">
                        <span class="text-zinc-300">${phase.name}</span>
                        <span class="font-mono text-zinc-400">${formatMoney(phase.cost)}</span>
                    </div>
                    <div class="h-2.5 bg-zinc-800 rounded-3xl overflow-hidden">
                        <div class="h-2.5 bg-gradient-to-r from-indigo-500 to-violet-500 rounded-3xl transition-all" style="width: ${percent}%"></div>
                    </div>
                `;
                container.appendChild(div);
            });
        }

        function renderTimeline(labels, outflows, cashEnds) {
            const tbody = document.getElementById('timeline-body');
            tbody.innerHTML = '';
            
            for (let i = 0; i < 30; i++) {
                const row = document.createElement('tr');
                row.className = `table-row ${cashEnds[i] < 0 ? 'bg-red-950/30' : ''}`;
                row.innerHTML = `
                    <td class="px-8 py-5 font-medium">${labels[i]}</td>
                    <td class="px-8 py-5 text-red-400 font-medium">−${formatMoney(outflows[i])}</td>
                    <td class="px-8 py-5 ${cashEnds[i] < 0 ? 'text-red-400' : 'text-emerald-400'} font-medium">${formatMoney(cashEnds[i])}</td>
                `;
                tbody.appendChild(row);
            }
        }

        function exportCSV() {
            const inp = getCurrentInputs();
            // Re-calculate to get accurate data
            const stampDuty = calculateStampDuty(inp.purchasePrice);
            const demoRaw = inp.surveyor + inp.structuralEngineer + inp.insurance + inp.dilapidationReport + inp.soilTest + inp.demolitionWorks + inp.makeGoodWorks;
            const totalDemo = Math.round(demoRaw * (1 + inp.contingency / 100));
            const preRaw = inp.architecturePlans + inp.engineeringPlans;
            const buildRaw = inp.slab + inp.frame + inp.roof + inp.internal + inp.finishing;
            const totalBuild = Math.round(buildRaw * (1 + inp.contingency / 100));
            
            const settlementMonth = Math.ceil(inp.settlementDays / 30);
            const monthlyRent = Math.round(inp.rentPerWeek * 52 / 12);
            const monthlyHolding = monthlyRent + inp.livingExpenses;
            const monthlyIncome = inp.djTint + inp.alexTint + inp.djRodeo;
            
            const purchaseLoan = Math.round(inp.purchasePrice * 0.8);
            const constructionLoan = Math.round(totalBuild * 0.8);
            const monthlyRate = inp.interestRate / 1200;
            let monthlyPI = 0;
            if (purchaseLoan + constructionLoan > 0) {
                const r = monthlyRate;
                const n = 360;
                monthlyPI = Math.round((purchaseLoan + constructionLoan) * r * Math.pow(1 + r, n) / (Math.pow(1 + r, n) - 1));
            }
            
            let cash = inp.prePurchaseCash;
            let csv = 'Month,Monthly Outflow,Cash on Hand\n';
            
            let currentDate = new Date(2026, 2, 1);
            
            const phaseOut = new Array(31).fill(0);
            if (settlementMonth <= 30) phaseOut[settlementMonth] += Math.round(inp.purchasePrice * 0.2 + stampDuty + inp.conveyancing);
            let demoM = settlementMonth + 1;
            for (let i = 0; i < 3; i++) {
                if (demoM + i <= 30) phaseOut[demoM + i] += Math.round(totalDemo / 3);
            }
            if (6 <= 30) phaseOut[6] += preRaw;
            if (7 <= 30) phaseOut[7] += Math.round(totalBuild * 0.2);
            
            for (let m = 1; m <= 30; m++) {
                let outflow = monthlyHolding;
                let interest = 0;
                if (m >= settlementMonth) interest += purchaseLoan * monthlyRate;
                if (m >= 7) interest += constructionLoan * monthlyRate;
                if (m >= 26) {
                    outflow += monthlyPI;
                } else {
                    outflow += interest;
                }
                outflow += phaseOut[m];
                
                const net = monthlyIncome - outflow;
                cash = Math.round(cash + net);
                
                const label = currentDate.toLocaleString('en-US', { month: 'short' }).toUpperCase() + ' ' + currentDate.getFullYear().toString().slice(2);
                csv += `${label},${Math.round(outflow)},${cash}\n`;
                currentDate.setMonth(currentDate.getMonth() + 1);
            }
            
            const blob = new Blob([csv], { type: 'text/csv' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'BurnTrack_Cashflow_Mar2026.csv';
            a.click();
            URL.revokeObjectURL(url);
        }

        function resetApp() {
            location.reload();
        }

        // Init everything
        function initializeApp() {
            console.log('%cBurnTrack v10 initialized successfully ✓', 'color:#6366f1; font-family:monospace; font-size:13px');
            
            // Render inputs
            renderInputs();
            
            // Initial calculation
            setTimeout(() => {
                calculateAndRenderAll();
            }, 180);
        }

        window.onload = initializeApp;
    </script>
</body>
</html>
