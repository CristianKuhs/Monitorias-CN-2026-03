<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Monitorias CN - Sistema de Avaliação de Atendimentos</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen',
                'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue', sans-serif;
            -webkit-font-smoothing: antialiased;
            -moz-osx-font-smoothing: grayscale;
            overflow-x: hidden;
        }
        ::-webkit-scrollbar {
            width: 10px;
        }
        ::-webkit-scrollbar-track {
            background: rgba(0, 0, 0, 0.1);
            border-radius: 10px;
        }
        ::-webkit-scrollbar-thumb {
            background: rgba(139, 92, 246, 0.5);
            border-radius: 10px;
        }
        .glass {
            backdrop-filter: blur(10px);
            -webkit-backdrop-filter: blur(10px);
        }
        @keyframes slideIn {
            from {
                opacity: 0;
                transform: translateY(10px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }
        .animate-in {
            animation: slideIn 0.5s ease-out;
        }
        .screen {
            display: none;
        }
        .screen.active {
            display: block;
            animation: slideIn 0.3s ease-out;
        }
        .icon {
            display: inline-block;
            font-size: inherit;
        }
    </style>
</head>
<body>
    <div id="app"></div>
    <script>
        // ============================================
        // DATA & CONFIG
        // ============================================
        const INITIAL_AGENTS = [
            'Charlote', 'Cristian', 'Eduardo', 'Emilly', 'Felipe', 'Fernando',
            'João', 'João Lucas', 'Kaua', 'Kauane', 'Pamela', 'Patrick',
            'Pedro', 'Renan', 'Richard', 'Thais', 'Vitória'
        ];
        const LIGACAO_RULES = [
            { id: 'lig_1', category: 'ABERTURA', action: 'Utilizou script inicial cordial?', weight: 10 },
            { id: 'lig_2', category: 'ABERTURA', action: 'Usou nome da empresa?', weight: 5 },
            { id: 'lig_3', category: 'ABERTURA', action: 'Confirmou dados do cliente?', weight: 10 },
            { id: 'lig_4', category: 'SONDAGEM E CONDUÇÃO', action: 'Sondou corretamente?', weight: 10 },
            { id: 'lig_5', category: 'SONDAGEM E CONDUÇÃO', action: 'Demonstrou empatia?', weight: 10 },
            { id: 'lig_6', category: 'SONDAGEM E CONDUÇÃO', action: 'Usou linguagem adequada?', weight: 5 },
            { id: 'lig_7', category: 'SONDAGEM E CONDUÇÃO', action: 'Informou ausência durante atendimento?', weight: 5 },
            { id: 'lig_8', category: 'CONHECIMENTO DOS PROCESSOS', action: 'Foi ágil na busca?', weight: 5 },
            { id: 'lig_9', category: 'CONHECIMENTO DOS PROCESSOS', action: 'Registrou corretamente?', weight: 10 },
            { id: 'lig_10', category: 'CONHECIMENTO DOS PROCESSOS', action: 'Orientações coerentes?', weight: 10 },
            { id: 'lig_11', category: 'FECHAMENTO', action: 'Confirmou compreensão?', weight: 5 },
            { id: 'lig_12', category: 'FECHAMENTO', action: 'Questionou necessidade adicional?', weight: 5 },
            { id: 'lig_13', category: 'FECHAMENTO', action: 'Passou protocolo?', weight: 5 },
            { id: 'lig_14', category: 'FECHAMENTO', action: 'Solicitou avaliação?', weight: 5 },
        ];
        const MATRIX_RULES = [
            { id: 'mat_1', category: 'ABERTURA', action: 'Utilizou script inicial cordial?', weight: 10 },
            { id: 'mat_2', category: 'ABERTURA', action: 'Confirmou dados do cliente?', weight: 10 },
            { id: 'mat_3', category: 'SONDAGEM E CONDUÇÃO', action: 'Sondou corretamente?', weight: 10 },
            { id: 'mat_4', category: 'SONDAGEM E CONDUÇÃO', action: 'Demonstrou empatia?', weight: 10 },
            { id: 'mat_5', category: 'SONDAGEM E CONDUÇÃO', action: 'Linguagem didática?', weight: 5 },
            { id: 'mat_6', category: 'SONDAGEM E CONDUÇÃO', action: 'Utilizou normas da linguagem corretamente?', weight: 10 },
            { id: 'mat_7', category: 'SONDAGEM E CONDUÇÃO', action: 'Não deixou cliente 15min sem resposta?', weight: 10 },
            { id: 'mat_8', category: 'CONHECIMENTO DOS PROCESSOS', action: 'Foi ágil na busca?', weight: 5 },
            { id: 'mat_9', category: 'CONHECIMENTO DOS PROCESSOS', action: 'Registrou corretamente?', weight: 10 },
            { id: 'mat_10', category: 'CONHECIMENTO DOS PROCESSOS', action: 'Orientações coerentes?', weight: 10 },
            { id: 'mat_11', category: 'FECHAMENTO', action: 'Confirmou compreensão?', weight: 5 },
            { id: 'mat_12', category: 'FECHAMENTO', action: 'Questionou necessidade adicional?', weight: 5 },
        ];
        // ============================================
        // STATE MANAGEMENT
        // ============================================
        let appState = {
            currentScreen: 'AGENT_SELECTION',
            agents: JSON.parse(localStorage.getItem('agents')) || INITIAL_AGENTS,
            monitorias: JSON.parse(localStorage.getItem('monitorias')) || [],
            selection: {},
            selectedRules: {},
            currentMonitoria: null,
            detailMonitoria: null,
        };
        function saveState() {
            localStorage.setItem('agents', JSON.stringify(appState.agents));
            localStorage.setItem('monitorias', JSON.stringify(appState.monitorias));
        }
        function getRulesByType(type) {
            return type === 'Ligação' ? LIGACAO_RULES : MATRIX_RULES;
        }
        function getScoreColor(score) {
            if (score >= 90) return { bg: 'bg-green-100', text: 'text-green-800', badge: 'bg-green-500', label: 'Excelente' };
            if (score >= 75) return { bg: 'bg-yellow-100', text: 'text-yellow-800', badge: 'bg-yellow-500', label: 'Bom' };
            if (score >= 60) return { bg: 'bg-orange-100', text: 'text-orange-800', badge: 'bg-orange-500', label: 'Atenção' };
            return { bg: 'bg-red-100', text: 'text-red-800', badge: 'bg-red-500', label: 'Crítico' };
        }
        function calculateScore(selectedRules, rules) {
            let score = 0;
            let issues = [];
            rules.forEach(rule => {
                if (selectedRules[rule.id] === true) {
                    score += rule.weight;
                } else if (selectedRules[rule.id] === false) {
                    issues.push(rule.action);
                }
            });
            return { score, issues };
        }
        function getRanking() {
            const ranking = appState.agents
                .map(agent => {
                    const agentMonitorias = appState.monitorias.filter(m => m.agent === agent);
                    if (agentMonitorias.length === 0) return null;
                    const average = agentMonitorias.reduce((a, b) => a + b.score, 0) / agentMonitorias.length;
                    return {
                        agent,
                        average: parseFloat(average.toFixed(1)),
                        count: agentMonitorias.length,
                    };
                })
                .filter(Boolean)
                .sort((a, b) => b.average - a.average);
            return ranking;
        }
        function getRecurringIssues(limit = 5) {
            const issuesMap = {};
            appState.monitorias.forEach(m => {
                if (m.issues) {
                    m.issues.forEach(issue => {
                        issuesMap[issue] = (issuesMap[issue] || 0) + 1;
                    });
                }
            });
            return Object.entries(issuesMap)
                .sort((a, b) => b[1] - a[1])
                .slice(0, limit)
                .map(([issue, count]) => ({ issue, count }));
        }
        // ============================================
        // RENDER FUNCTIONS
        // ============================================
        function render() {
            const app = document.getElementById('app');
            if (appState.currentScreen === 'AGENT_SELECTION') {
                app.innerHTML = renderAgentSelection();
                attachAgentSelectionListeners();
            } else if (appState.currentScreen === 'INITIAL_INFO') {
                app.innerHTML = renderInitialInfo();
                attachInitialInfoListeners();
            } else if (appState.currentScreen === 'EVALUATION') {
                app.innerHTML = renderEvaluation();
                attachEvaluationListeners();
            } else if (appState.currentScreen === 'RESULT') {
                app.innerHTML = renderResult();
                attachResultListeners();
            } else if (appState.currentScreen === 'HISTORY') {
                app.innerHTML = renderHistory();
                attachHistoryListeners();
            } else if (appState.currentScreen === 'DETAIL') {
                app.innerHTML = renderDetail();
                attachDetailListeners();
            } else if (appState.currentScreen === 'DASHBOARD') {
                app.innerHTML = renderDashboard();
                attachDashboardListeners();
            }
        }
        function renderAgentSelection() {
            return `
                <div class="min-h-screen bg-gradient-to-br from-blue-600 via-purple-600 to-purple-700 flex items-center justify-center p-4">
                    <div class="absolute inset-0 overflow-hidden pointer-events-none">
                        <div class="absolute -top-40 -right-40 w-80 h-80 bg-white/10 rounded-full blur-3xl"></div>
                        <div class="absolute -bottom-40 -left-40 w-80 h-80 bg-white/10 rounded-full blur-3xl"></div>
                    </div>
                    <div class="relative z-10 w-full max-w-2xl animate-in">
                        <div class="glass rounded-3xl p-8 md:p-12 border border-white/20 shadow-2xl bg-white/10">
                            <div class="text-center mb-12">
                                <h1 class="text-4xl md:text-5xl font-bold text-white mb-3">Monitorias CN</h1>
                                <p class="text-white/80 text-lg">Sistema de Avaliação de Atendimentos</p>
                            </div>
                            <div class="mb-8">
                                <label class="block text-white font-semibold mb-4 text-lg">👤 Selecione o Agente</label>
                                <div class="grid grid-cols-2 md:grid-cols-3 gap-3 mb-4" id="agentGrid">
                                    ${appState.agents.map(agent => `
                                        <div class="relative">
                                            <button class="agent-btn w-full py-3 px-4 rounded-xl font-medium transition-all duration-300 bg-white/20 text-white hover:bg-white/30 ${appState.selection.agent === agent ? 'bg-white text-purple-600 scale-105' : ''}" data-agent="${agent}">
                                                ${agent}
                                            </button>
                                            <button class="delete-agent absolute -top-2 -right-2 bg-red-500 rounded-full p-1 opacity-0 hover:opacity-100 transition-opacity duration-300 hover:bg-red-600" data-agent="${agent}" title="Excluir agente">
                                                ✕
                                            </button>
                                        </div>
                                    `).join('')}
                                </div>
                                <button id="addAgentBtn" class="w-full py-3 px-4 rounded-xl border-2 border-dashed border-white/40 text-white hover:bg-white/10 transition-all duration-300 flex items-center justify-center gap-2 font-medium">
                                    ➕ Adicionar novo agente
                                </button>
                                <div id="addAgentForm" style="display: none;" class="mt-4 flex gap-2">
                                    <input type="text" id="newAgentName" placeholder="Nome do novo agente" class="flex-1 px-4 py-3 rounded-xl bg-white/20 text-white placeholder-white/50 border border-white/30 focus:outline-none focus:ring-2 focus:ring-white">
                                    <button id="confirmAddAgent" class="px-6 py-3 bg-white text-purple-600 rounded-xl font-semibold hover:bg-opacity-90 transition-all duration-300">Adicionar</button>
                                </div>
                            </div>
                            <div class="mb-8">
                                <label class="block text-white font-semibold mb-4 text-lg">📞 Tipo de Atendimento</label>
                                <div class="grid grid-cols-2 gap-4">
                                    <button class="type-btn py-6 px-6 rounded-2xl font-bold text-lg transition-all duration-300 bg-white/20 text-white hover:bg-white/30 ${appState.selection.type === 'Ligação' ? 'bg-white text-purple-600 scale-105' : ''}" data-type="Ligação">
                                        📞 Ligação
                                    </button>
                                    <button class="type-btn py-6 px-6 rounded-2xl font-bold text-lg transition-all duration-300 bg-white/20 text-white hover:bg-white/30 ${appState.selection.type === 'Matrix' ? 'bg-white text-purple-600 scale-105' : ''}" data-type="Matrix">
                                        💬 Matrix
                                    </button>
                                </div>
                            </div>
                            <button id="nextBtn" class="w-full py-4 px-6 rounded-xl font-bold text-lg flex items-center justify-center gap-2 transition-all duration-300 ${appState.selection.agent && appState.selection.type ? 'bg-white text-purple-600 hover:shadow-lg hover:scale-105 cursor-pointer' : 'bg-white/20 text-white/50 cursor-not-allowed'}">
                                Próxima Etapa ▶
                            </button>
                        </div>
                    </div>
                </div>
            `;
        }
        function renderInitialInfo() {
            const today = new Date().toISOString().split('T')[0];
            return `
                <div class="min-h-screen bg-gradient-to-br from-blue-600 via-purple-600 to-purple-700 flex items-center justify-center p-4">
                    <div class="absolute inset-0 overflow-hidden pointer-events-none">
                        <div class="absolute -top-40 -right-40 w-80 h-80 bg-white/10 rounded-full blur-3xl"></div>
                        <div class="absolute -bottom-40 -left-40 w-80 h-80 bg-white/10 rounded-full blur-3xl"></div>
                    </div>
                    <div class="relative z-10 w-full max-w-2xl animate-in">
                        <div class="glass rounded-3xl p-8 md:p-12 border border-white/20 shadow-2xl bg-white/10">
                            <div class="mb-8">
                                <h2 class="text-2xl font-bold text-white">${appState.selection.type === 'Ligação' ? '📞' : '💬'} ${appState.selection.type}</h2>
                                <p class="text-white/70 mt-2">Agente: <strong>${appState.selection.agent}</strong></p>
                            </div>
                            <div class="space-y-6 mb-8">
                                <div>
                                    <label class="block text-white font-semibold mb-3">👤 Nome do Cliente</label>
                                    <input type="text" id="clientName" placeholder="Digite o nome do cliente" class="w-full px-4 py-4 rounded-xl bg-white/20 text-white placeholder-white/50 border border-white/30 focus:outline-none focus:ring-2 focus:ring-white transition-all duration-300">
                                </div>
                                <div>
                                    <label class="block text-white font-semibold mb-3">📅 Data do Atendimento</label>
                                    <input type="date" id="date" value="${today}" class="w-full px-4 py-4 rounded-xl bg-white/20 text-white border border-white/30 focus:outline-none focus:ring-2 focus:ring-white transition-all duration-300">
                                </div>
                                <div>
                                    <label class="block text-white font-semibold mb-3">🔐 Protocolo do Atendimento</label>
                                    <input type="text" id="protocol" placeholder="Digite o número do protocolo" class="w-full px-4 py-4 rounded-xl bg-white/20 text-white placeholder-white/50 border border-white/30 focus:outline-none focus:ring-2 focus:ring-white transition-all duration-300">
                                </div>
                            </div>
                            <div class="flex gap-4 pt-6">
                                <button id="backBtn" class="flex-1 py-4 px-6 rounded-xl border-2 border-white/30 text-white font-bold hover:bg-white/10 transition-all duration-300 flex items-center justify-center gap-2">
                                    ◀ Voltar
                                </button>
                                <button id="evaluateBtn" class="flex-1 py-4 px-6 rounded-xl bg-white text-purple-600 font-bold hover:shadow-lg hover:scale-105 transition-all duration-300 flex items-center justify-center gap-2">
                                    Avaliar ▶
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            `;
        }
        function renderEvaluation() {
            const rules = getRulesByType(appState.selection.type);
            const categories = [...new Set(rules.map(r => r.category))];
            let totalScore = 0;
            let respondedCount = 0;
            rules.forEach(rule => {
                if (appState.selectedRules[rule.id] !== undefined) {
                    respondedCount++;
                    if (appState.selectedRules[rule.id]) {
                        totalScore += rule.weight;
                    }
                }
            });
            const badgeColor = totalScore >= 90 ? 'bg-green-500' : totalScore >= 75 ? 'bg-yellow-500' : totalScore >= 60 ? 'bg-orange-500' : 'bg-red-500';
            return `
                <div class="min-h-screen bg-gradient-to-br from-blue-50 to-purple-50">
                    <div class="sticky top-0 z-20 glass bg-white/80 border-b border-white/20 py-6">
                        <div class="max-w-7xl mx-auto px-4 md:px-8">
                            <div class="flex flex-col md:flex-row items-start md:items-center justify-between">
                                <div>
                                    <p class="text-purple-600 font-semibold text-sm mb-1">MONITORIA DE ATENDIMENTO</p>
                                    <h1 class="text-3xl md:text-4xl font-bold text-gray-800">${appState.selection.type === 'Ligação' ? '📞' : '💬'} ${appState.selection.type}</h1>
                                </div>
                                <div class="mt-4 md:mt-0 flex items-center gap-3">
                                    <div class="${badgeColor} text-white font-bold text-center rounded-full w-20 h-20 flex items-center justify-center text-2xl shadow-lg">
                                        ${totalScore}
                                    </div>
                                </div>
                            </div>
                            <div class="mt-4">
                                <div class="flex justify-between items-center mb-2">
                                    <span class="text-sm font-semibold text-gray-700">Progresso: ${respondedCount}/${rules.length}</span>
                                    <span class="text-sm font-semibold text-purple-600">${Math.round((respondedCount / rules.length) * 100)}%</span>
                                </div>
                                <div class="w-full bg-gray-200 rounded-full h-3 overflow-hidden">
                                    <div class="${badgeColor} h-full transition-all duration-500" style="width: ${Math.min(totalScore, 100)}%"></div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <main class="max-w-7xl mx-auto px-4 md:px-8 py-8">
                        ${categories.map(category => {
                            const categoryRules = rules.filter(r => r.category === category);
                            return `
                                <div class="mb-8">
                                    <h3 class="text-lg font-bold text-gray-800 mb-4">${category}</h3>
                                    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                                        ${categoryRules.map(rule => `
                                            <div class="glass rounded-2xl p-6 border border-white/20 hover:bg-white/70 transition-all duration-300">
                                                <div class="flex items-start justify-between mb-4">
                                                    <div class="flex-1">
                                                        <p class="text-xs font-semibold text-purple-600 mb-1">${rule.category}</p>
                                                        <p class="text-sm font-medium text-gray-800">${rule.action}</p>
                                                    </div>
                                                    <span class="bg-purple-100 text-purple-700 px-3 py-1 rounded-full text-sm font-semibold ml-4">${rule.weight}pts</span>
                                                </div>
                                                <div class="flex gap-4">
                                                    <label class="flex items-center gap-2 cursor-pointer flex-1">
                                                        <input type="radio" name="rule_${rule.id}" value="yes" class="rule-radio w-4 h-4 cursor-pointer" data-rule="${rule.id}" ${appState.selectedRules[rule.id] === true ? 'checked' : ''}>
                                                        <span class="text-sm text-gray-700">✓ Sim</span>
                                                    </label>
                                                    <label class="flex items-center gap-2 cursor-pointer flex-1">
                                                        <input type="radio" name="rule_${rule.id}" value="no" class="rule-radio w-4 h-4 cursor-pointer" data-rule="${rule.id}" ${appState.selectedRules[rule.id] === false ? 'checked' : ''}>
                                                        <span class="text-sm text-gray-700">✗ Não</span>
                                                    </label>
                                                </div>
                                            </div>
                                        `).join('')}
                                    </div>
                                </div>
                            `;
                        }).join('')}
                        <div class="flex gap-4 mt-12 pt-8 border-t border-gray-200">
                            <button id="backEvalBtn" class="flex-1 py-4 px-6 rounded-xl border-2 border-gray-300 text-gray-700 font-bold hover:bg-gray-50 transition-all duration-300">
                                ◀ Voltar
                            </button>
                            <button id="submitBtn" class="${respondedCount === rules.length ? 'bg-green-500 text-white hover:bg-green-600 hover:shadow-lg hover:scale-105 cursor-pointer' : 'bg-gray-300 text-gray-500 cursor-not-allowed'} flex-1 py-4 px-6 rounded-xl font-bold flex items-center justify-center gap-2 transition-all duration-300 text-lg">
                                ✓ Finalizar
                            </button>
                        </div>
                    </main>
                </div>
            `;
        }
        function renderResult() {
            const m = appState.currentMonitoria;
            const color = getScoreColor(m.score);
            return `
                <div class="min-h-screen bg-gradient-to-br from-blue-50 to-purple-50">
                    <div class="glass bg-white/80 border-b border-white/20 py-8">
                        <div class="max-w-4xl mx-auto px-4 md:px-8">
                            <p class="text-purple-600 font-semibold text-sm mb-2">RESULTADO DA MONITORIA</p>
                            <h1 class="text-3xl md:text-4xl font-bold text-gray-800">Avaliação Finalizada</h1>
                        </div>
                    </div>
                    <main class="max-w-4xl mx-auto px-4 md:px-8 py-12">
                        <div class="glass rounded-3xl p-8 md:p-12 border border-white/20 shadow-2xl mb-8 ${color.bg}">
                            <div class="flex flex-col md:flex-row items-center gap-8">
                                <div class="${color.badge} text-white rounded-full w-40 h-40 flex items-center justify-center flex-shrink-0 shadow-lg">
                                    <div class="text-center">
                                        <p class="text-6xl font-bold">${m.score}</p>
                                        <p class="text-xl mt-2">pontos</p>
                                    </div>
                                </div>
                                <div class="flex-1">
                                    <p class="${color.text} text-5xl font-bold mb-3">${color.label}</p>
                                    <div class="w-full bg-gray-200 rounded-full h-3 mb-6">
                                        <div class="${color.badge} h-full" style="width: ${m.score}%"></div>
                                    </div>
                                    <div class="grid grid-cols-2 gap-4">
                                        <div>
                                            <p class="text-sm text-gray-600 mb-1">Tipo</p>
                                            <p class="text-lg font-bold text-gray-800">${m.type === 'Ligação' ? '📞' : '💬'} ${m.type}</p>
                                        </div>
                                        <div>
                                            <p class="text-sm text-gray-600 mb-1">Agente</p>
                                            <p class="text-lg font-bold text-gray-800">${m.agent}</p>
                                        </div>
                                        <div>
                                            <p class="text-sm text-gray-600 mb-1">Cliente</p>
                                            <p class="text-lg font-bold text-gray-800">${m.clientName}</p>
                                        </div>
                                        <div>
                                            <p class="text-sm text-gray-600 mb-1">Protocolo</p>
                                            <p class="text-lg font-bold text-gray-800 font-mono">${m.protocol}</p>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        ${m.issues && m.issues.length > 0 ? `
                            <div class="mb-8">
                                <h3 class="text-xl font-bold text-gray-800 mb-4">⚠️ Pontos para Melhoria (${m.issues.length})</h3>
                                <div class="glass rounded-2xl p-6 border border-orange-200/50 bg-orange-50/40">
                                    <ul class="space-y-2">
                                        ${m.issues.map((issue, idx) => `
                                            <li class="flex items-start gap-3">
                                                <span class="text-orange-500 font-bold mt-1">•</span>
                                                <span class="text-gray-700">${issue}</span>
                                            </li>
                                        `).join('')}
                                    </ul>
                                </div>
                            </div>
                        ` : ''}
                        <div class="flex gap-4 mt-12 pt-8 border-t border-gray-200">
                            <button id="historyBtn" class="flex-1 py-4 px-6 rounded-xl border-2 border-purple-300 text-purple-600 font-bold hover:bg-purple-50 transition-all duration-300">
                                📊 Ver Histórico
                            </button>
                            <button id="newMonitoriaBtn" class="flex-1 py-4 px-6 rounded-xl bg-purple-600 text-white font-bold hover:bg-purple-700 transition-all duration-300">
                                ➕ Nova Monitoria
                            </button>
                        </div>
                    </main>
                </div>
            `;
        }
        function renderHistory() {
            const monitorias = appState.monitorias;
            return `
                <div class="min-h-screen bg-gradient-to-br from-blue-50 to-purple-50">
                    <div class="sticky top-0 z-20 glass bg-white/80 border-b border-white/20 py-8">
                        <div class="max-w-7xl mx-auto px-4 md:px-8">
                            <div class="flex flex-col md:flex-row items-start md:items-center justify-between">
                                <div>
                                    <p class="text-purple-600 font-semibold text-sm mb-2">HISTÓRICO</p>
                                    <h1 class="text-3xl md:text-4xl font-bold text-gray-800">📋 Monitorias Finalizadas</h1>
                                    <p class="text-gray-600 mt-2">Total: <strong>${monitorias.length}</strong></p>
                                </div>
                                <div class="flex gap-3 mt-4 md:mt-0 w-full md:w-auto">
                                    <button id="dashboardBtn" class="flex-1 md:flex-none py-3 px-6 rounded-xl bg-purple-600 text-white font-bold hover:bg-purple-700 transition-all duration-300">
                                        📊 Dashboard
                                    </button>
                                    <button id="newBtn" class="flex-1 md:flex-none py-3 px-6 rounded-xl border-2 border-purple-600 text-purple-600 font-bold hover:bg-purple-50 transition-all duration-300">
                                        ➕ Nova
                                    </button>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="max-w-7xl mx-auto px-4 md:px-8 pb-12 pt-8">
                        ${monitorias.length === 0 ? `
                            <div class="glass rounded-2xl p-12 border border-white/20 text-center">
                                <p class="text-3xl mb-4">📭</p>
                                <p class="text-gray-600 text-lg">Nenhuma monitoria registrada</p>
                            </div>
                        ` : `
                            <div class="overflow-x-auto rounded-2xl border border-gray-200">
                                <table class="w-full">
                                    <thead class="glass bg-gray-50 border-b border-gray-200">
                                        <tr>
                                            <th class="px-6 py-4 text-left text-xs font-semibold text-gray-700 uppercase">Agente</th>
                                            <th class="px-6 py-4 text-left text-xs font-semibold text-gray-700 uppercase">Tipo</th>
                                            <th class="px-6 py-4 text-left text-xs font-semibold text-gray-700 uppercase">Cliente</th>
                                            <th class="px-6 py-4 text-left text-xs font-semibold text-gray-700 uppercase">Data</th>
                                            <th class="px-6 py-4 text-left text-xs font-semibold text-gray-700 uppercase">Protocolo</th>
                                            <th class="px-6 py-4 text-left text-xs font-semibold text-gray-700 uppercase">Nota</th>
                                            <th class="px-6 py-4 text-left text-xs font-semibold text-gray-700 uppercase">Status</th>
                                            <th class="px-6 py-4 text-center text-xs font-semibold text-gray-700 uppercase">Ação</th>
                                        </tr>
                                    </thead>
                                    <tbody>
                                        ${monitorias.map((m, idx) => {
                                            const color = getScoreColor(m.score);
                                            return `
                                                <tr class="${idx % 2 === 0 ? 'bg-white' : 'bg-gray-50/50'} border-b border-gray-200 hover:bg-gray-50">
                                                    <td class="px-6 py-4"><span class="font-semibold text-gray-800">${m.agent}</span></td>
                                                    <td class="px-6 py-4"><span class="text-gray-700">${m.type === 'Ligação' ? '📞' : '💬'} ${m.type}</span></td>
                                                    <td class="px-6 py-4"><span class="text-gray-700">${m.clientName}</span></td>
                                                    <td class="px-6 py-4"><span class="text-gray-600 text-sm">${new Date(m.date).toLocaleDateString('pt-BR')}</span></td>
                                                    <td class="px-6 py-4"><span class="font-mono text-sm text-gray-600">${m.protocol}</span></td>
                                                    <td class="px-6 py-4"><span class="font-bold text-lg text-gray-800">${m.score}</span></td>
                                                    <td class="px-6 py-4"><span class="inline-block px-3 py-1 rounded-full text-sm font-semibold text-white ${color.badge}">${color.label}</span></td>
                                                    <td class="px-6 py-4 text-center">
                                                        <button class="view-detail inline-flex items-center justify-center w-10 h-10 rounded-lg bg-purple-100 hover:bg-purple-200 text-purple-600 transition-all duration-300" data-id="${m.id}">
                                                            👁️
                                                        </button>
                                                    </td>
                                                </tr>
                                            `;
                                        }).join('')}
                                    </tbody>
                                </table>
                            </div>
                        `}
                    </div>
                </div>
            `;
        }
        function renderDetail() {
            const m = appState.detailMonitoria;
            const rules = getRulesByType(m.type);
            const categories = [...new Set(rules.map(r => r.category))];
            const color = getScoreColor(m.score);
            return `
                <div class="min-h-screen bg-gradient-to-br from-blue-50 to-purple-50">
                    <div class="sticky top-0 z-20 glass bg-white/80 border-b border-white/20 py-6">
                        <div class="max-w-7xl mx-auto px-4 md:px-8">
                            <button id="backDetailBtn" class="flex items-center gap-2 text-purple-600 font-semibold mb-4 hover:text-purple-700">
                                ◀ Voltar
                            </button>
                            <div class="flex flex-col md:flex-row items-start md:items-center justify-between">
                                <div>
                                    <h1 class="text-3xl md:text-4xl font-bold text-gray-800">${m.type === 'Ligação' ? '📞' : '💬'} ${m.type}</h1>
                                </div>
                                <div class="glass rounded-2xl p-6 border border-white/20 mt-4 md:mt-0">
                                    <div class="flex items-center gap-4">
                                        <div class="${color.badge} text-white rounded-full w-24 h-24 flex items-center justify-center font-bold text-3xl">
                                            ${m.score}
                                        </div>
                                        <div>
                                            <p class="${color.text} text-lg font-bold">${color.label}</p>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="max-w-7xl mx-auto px-4 md:px-8 py-8">
                        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 mb-8">
                            <div class="glass rounded-2xl p-6 border border-white/20">
                                <p class="text-sm text-gray-600 mb-2">Agente</p>
                                <p class="text-xl font-bold text-gray-800">${m.agent}</p>
                            </div>
                            <div class="glass rounded-2xl p-6 border border-white/20">
                                <p class="text-sm text-gray-600 mb-2">Cliente</p>
                                <p class="text-xl font-bold text-gray-800">${m.clientName}</p>
                            </div>
                            <div class="glass rounded-2xl p-6 border border-white/20">
                                <p class="text-sm text-gray-600 mb-2">Data</p>
                                <p class="text-xl font-bold text-gray-800">${new Date(m.date).toLocaleDateString('pt-BR')}</p>
                            </div>
                            <div class="glass rounded-2xl p-6 border border-white/20">
                                <p class="text-sm text-gray-600 mb-2">Protocolo</p>
                                <p class="text-xl font-bold text-gray-800 font-mono">${m.protocol}</p>
                            </div>
                        </div>
                        ${categories.map(category => {
                            const categoryRules = rules.filter(r => r.category === category);
                            return `
                                <div class="mb-12">
                                    <h3 class="text-lg font-bold text-gray-800 mb-4">${category}</h3>
                                    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                                        ${categoryRules.map(rule => {
                                            const isYes = m.selectedRules[rule.id] === true;
                                            const isNo = m.selectedRules[rule.id] === false;
                                            return `
                                                <div class="glass rounded-2xl p-6 border ${isYes ? 'border-green-200/50 bg-green-50/40' : isNo ? 'border-red-200/50 bg-red-50/40' : 'border-white/20'}">
                                                    <div class="flex items-start justify-between mb-3">
                                                        <div class="flex-1">
                                                            <p class="text-xs font-semibold text-purple-600 mb-1">${category}</p>
                                                            <p class="text-sm font-medium text-gray-800">${rule.action}</p>
                                                        </div>
                                                        <span class="bg-purple-100 text-purple-700 px-3 py-1 rounded-full text-sm font-semibold ml-2">${rule.weight}pts</span>
                                                    </div>
                                                    <div class="flex items-center gap-2">
                                                        ${isYes ? '<span class="text-2xl">✓</span><span class="text-green-700 font-semibold">Respeitado</span>' : isNo ? '<span class="text-2xl">✗</span><span class="text-red-700 font-semibold">Não respeitado</span>' : ''}
                                                    </div>
                                                </div>
                                            `;
                                        }).join('')}
                                    </div>
                                </div>
                            `;
                        }).join('')}
                        ${m.issues && m.issues.length > 0 ? `
                            <div class="mt-12">
                                <h3 class="text-lg font-bold text-gray-800 mb-4">⚠️ Pontos para Melhoria</h3>
                                <div class="glass rounded-2xl p-6 border border-orange-200/50 bg-orange-50/40">
                                    <ul class="space-y-3">
                                        ${m.issues.map((issue, idx) => `
                                            <li class="flex items-start gap-3">
                                                <span class="text-orange-500 font-bold mt-1">→</span>
                                                <span class="text-gray-700">${issue}</span>
                                            </li>
                                        `).join('')}
                                    </ul>
                                </div>
                            </div>
                        ` : ''}
                    </div>
                </div>
            `;
        }
        function renderDashboard() {
            const ranking = getRanking();
            const recurringIssues = getRecurringIssues();
            const monitorias = appState.monitorias;
            let stats = {
                totalMonitorias: monitorias.length,
                averageScore: 0,
                excellentCount: 0,
                goodCount: 0,
                attentionCount: 0,
                criticalCount: 0,
            };
            if (monitorias.length > 0) {
                const scores = monitorias.map(m => m.score);
                stats.averageScore = parseFloat((scores.reduce((a, b) => a + b, 0) / scores.length).toFixed(1));
                stats.excellentCount = scores.filter(s => s >= 90).length;
                stats.goodCount = scores.filter(s => s >= 75 && s < 90).length;
                stats.attentionCount = scores.filter(s => s >= 60 && s < 75).length;
                stats.criticalCount = scores.filter(s => s < 60).length;
            }
            return `
                <div class="min-h-screen bg-gradient-to-br from-blue-50 to-purple-50">
                    <div class="sticky top-0 z-20 glass bg-white/80 border-b border-white/20 py-8">
                        <div class="max-w-7xl mx-auto px-4 md:px-8">
                            <button id="backDashBtn" class="flex items-center gap-2 text-purple-600 font-semibold mb-4 hover:text-purple-700">
                                ◀ Voltar
                            </button>
                            <div class="flex flex-col md:flex-row items-start md:items-center justify-between">
                                <div>
                                    <p class="text-purple-600 font-semibold text-sm mb-2">INTELIGÊNCIA DE DADOS</p>
                                    <h1 class="text-3xl md:text-4xl font-bold text-gray-800">📊 Dashboard de Desempenho</h1>
                                </div>
                                <button id="newMonFromDashBtn" class="mt-4 md:mt-0 py-3 px-6 rounded-xl bg-purple-600 text-white font-bold hover:bg-purple-700 transition-all duration-300">
                                    ➕ Nova Monitoria
                                </button>
                            </div>
                        </div>
                    </div>
                    <main class="max-w-7xl mx-auto px-4 md:px-8 py-12">
                        ${monitorias.length === 0 ? `
                            <div class="glass rounded-2xl p-12 border border-white/20 text-center">
                                <p class="text-3xl mb-4">📭</p>
                                <p class="text-gray-600 text-lg">Nenhuma monitoria registrada</p>
                            </div>
                        ` : `
                            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-12">
                                <div class="glass rounded-2xl p-6 border border-white/20">
                                    <p class="text-sm text-gray-600 mb-2">Monitorias Realizadas</p>
                                    <p class="text-3xl font-bold text-gray-800">${stats.totalMonitorias}</p>
                                </div>
                                <div class="glass rounded-2xl p-6 border border-white/20">
                                    <p class="text-sm text-gray-600 mb-2">Nota Média Geral</p>
                                    <p class="text-3xl font-bold text-gray-800">${stats.averageScore}</p>
                                </div>
                                <div class="glass rounded-2xl p-6 border border-white/20">
                                    <p class="text-sm text-gray-600 mb-2">Atendimento Excelente</p>
                                    <p class="text-3xl font-bold text-green-600">${stats.excellentCount}</p>
                                </div>
                                <div class="glass rounded-2xl p-6 border border-white/20">
                                    <p class="text-sm text-gray-600 mb-2">Precisam Atenção</p>
                                    <p class="text-3xl font-bold text-red-600">${stats.attentionCount + stats.criticalCount}</p>
                                </div>
                            </div>
                            ${ranking.length > 0 ? `
                                <div class="mb-12">
                                    <h2 class="text-2xl font-bold text-gray-800 mb-8">🏆 Ranking Geral</h2>
                                    <div class="grid grid-cols-1 md:grid-cols-3 gap-8 mb-12">
                                        ${ranking.slice(0, 3).map((agent, idx) => {
                                            const position = idx + 1;
                                            const medals = ['🥇', '🥈', '🥉'];
                                            const heights = ['h-32', 'h-24', 'h-20'];
                                            return `
                                                <div class="flex flex-col items-center">
                                                    <div class="text-4xl mb-2">${medals[idx]}</div>
                                                    <div class="bg-gradient-to-t from-purple-500 to-purple-300 text-white font-bold text-2xl pb-4 rounded-t-2xl flex items-end justify-center ${heights[idx]} w-32 shadow-lg">
                                                        ${position}º
                                                    </div>
                                                    <div class="glass rounded-2xl p-4 mt-2 border border-white/20 w-32 text-center">
                                                        <p class="font-semibold text-gray-800 truncate">${agent.agent}</p>
                                                        <p class="text-sm text-purple-600 font-bold">${agent.average}</p>
                                                        <p class="text-xs text-gray-600">${agent.count} monitoria${agent.count > 1 ? 's' : ''}</p>
                                                    </div>
                                                </div>
                                            `;
                                        }).join('')}
                                    </div>
                                    ${ranking.length > 3 ? `
                                        <div class="glass rounded-2xl border border-white/20 overflow-hidden">
                                            <table class="w-full">
                                                <thead class="glass bg-gray-50 border-b border-gray-200">
                                                    <tr>
                                                        <th class="px-6 py-4 text-left text-sm font-semibold text-gray-700">Posição</th>
                                                        <th class="px-6 py-4 text-left text-sm font-semibold text-gray-700">Agente</th>
                                                        <th class="px-6 py-4 text-center text-sm font-semibold text-gray-700">Média</th>
                                                        <th class="px-6 py-4 text-center text-sm font-semibold text-gray-700">Monitorias</th>
                                                    </tr>
                                                </thead>
                                                <tbody>
                                                    ${ranking.map((agent, idx) => `
                                                        <tr class="${idx % 2 === 0 ? 'bg-white' : 'bg-gray-50/50'} border-b border-gray-200">
                                                            <td class="px-6 py-4 font-bold text-gray-800">${idx + 1}º</td>
                                                            <td class="px-6 py-4 font-semibold text-gray-800">${agent.agent}</td>
                                                            <td class="px-6 py-4 text-center">
                                                                <span class="inline-block px-3 py-1 rounded-full bg-purple-100 text-purple-700 font-bold">${agent.average}</span>
                                                            </td>
                                                            <td class="px-6 py-4 text-center text-gray-700">${agent.count}</td>
                                                        </tr>
                                                    `).join('')}
                                                </tbody>
                                            </table>
                                        </div>
                                    ` : ''}
                                </div>
                            ` : ''}
                            <div class="grid grid-cols-2 md:grid-cols-4 gap-4 mt-12">
                                <div class="glass rounded-2xl p-6 border border-green-200/50 bg-green-50/40 text-center">
                                    <p class="text-3xl mb-2">🟢</p>
                                    <p class="text-sm text-gray-600 mb-1">Excelente</p>
                                    <p class="text-2xl font-bold text-green-700">${stats.excellentCount}</p>
                                </div>
                                <div class="glass rounded-2xl p-6 border border-yellow-200/50 bg-yellow-50/40 text-center">
                                    <p class="text-3xl mb-2">🟡</p>
                                    <p class="text-sm text-gray-600 mb-1">Bom</p>
                                    <p class="text-2xl font-bold text-yellow-700">${stats.goodCount}</p>
                                </div>
                                <div class="glass rounded-2xl p-6 border border-orange-200/50 bg-orange-50/40 text-center">
                                    <p class="text-3xl mb-2">🟠</p>
                                    <p class="text-sm text-gray-600 mb-1">Atenção</p>
                                    <p class="text-2xl font-bold text-orange-700">${stats.attentionCount}</p>
                                </div>
                                <div class="glass rounded-2xl p-6 border border-red-200/50 bg-red-50/40 text-center">
                                    <p class="text-3xl mb-2">🔴</p>
                                    <p class="text-sm text-gray-600 mb-1">Crítico</p>
                                    <p class="text-2xl font-bold text-red-700">${stats.criticalCount}</p>
                                </div>
                            </div>
                            ${recurringIssues.length > 0 ? `
                                <div class="mt-12">
                                    <h3 class="text-lg font-bold text-gray-800 mb-6">⚠️ Erros Mais Recorrentes</h3>
                                    <div class="glass rounded-2xl p-6 border border-white/20">
                                        <div class="space-y-4">
                                            ${recurringIssues.map((item, idx) => {
                                                const maxCount = Math.max(...recurringIssues.map(i => i.count));
                                                return `
                                                    <div class="flex items-center gap-4">
                                                        <div class="flex-1">
                                                            <p class="text-sm text-gray-700 mb-2">${item.issue}</p>
                                                            <div class="w-full bg-gray-200 rounded-full h-2">
                                                                <div class="bg-red-500 h-2 rounded-full" style="width: ${(item.count / maxCount) * 100}%"></div>
                                                            </div>
                                                        </div>
                                                        <span class="font-bold text-red-600 w-12 text-right">${item.count}x</span>
                                                    </div>
                                                `;
                                            }).join('')}
                                        </div>
                                    </div>
                                </div>
                            ` : ''}
                        `}
                    </main>
                </div>
            `;
        }
        // ============================================
        // EVENT LISTENERS
        // ============================================
        function attachAgentSelectionListeners() {
            document.querySelectorAll('.agent-btn').forEach(btn => {
                btn.addEventListener('click', () => {
                    appState.selection.agent = btn.dataset.agent;
                    render();
                });
            });
            document.querySelectorAll('.delete-agent').forEach(btn => {
                btn.addEventListener('click', (e) => {
                    e.preventDefault();
                    const agent = btn.dataset.agent;
                    if (confirm(`Excluir agente ${agent}?`)) {
                        appState.agents = appState.agents.filter(a => a !== agent);
                        if (appState.selection.agent === agent) appState.selection.agent = '';
                        saveState();
                        render();
                    }
                });
            });
            document.getElementById('addAgentBtn').addEventListener('click', () => {
                document.getElementById('addAgentForm').style.display = 'flex';
            });
            document.getElementById('confirmAddAgent').addEventListener('click', () => {
                const name = document.getElementById('newAgentName').value.trim();
                if (name && !appState.agents.includes(name)) {
                    appState.agents.push(name);
                    appState.agents.sort();
                    document.getElementById('newAgentName').value = '';
                    document.getElementById('addAgentForm').style.display = 'none';
                    saveState();
                    render();
                }
            });
            document.getElementById('newAgentName').addEventListener('keypress', (e) => {
                if (e.key === 'Enter') document.getElementById('confirmAddAgent').click();
            });
            document.querySelectorAll('.type-btn').forEach(btn => {
                btn.addEventListener('click', () => {
                    appState.selection.type = btn.dataset.type;
                    render();
                });
            });
            document.getElementById('nextBtn').addEventListener('click', () => {
                if (appState.selection.agent && appState.selection.type) {
                    appState.currentScreen = 'INITIAL_INFO';
                    render();
                }
            });
        }
        function attachInitialInfoListeners() {
            document.getElementById('backBtn').addEventListener('click', () => {
                appState.selection = {};
                appState.currentScreen = 'AGENT_SELECTION';
                render();
            });
            document.getElementById('evaluateBtn').addEventListener('click', () => {
                const clientName = document.getElementById('clientName').value.trim();
                const date = document.getElementById('date').value;
                const protocol = document.getElementById('protocol').value.trim();
                if (clientName && date && protocol) {
                    appState.selection.clientName = clientName;
                    appState.selection.date = date;
                    appState.selection.protocol = protocol;
                    appState.selectedRules = {};
                    appState.currentScreen = 'EVALUATION';
                    render();
                } else {
                    alert('Preencha todos os campos!');
                }
            });
        }
        function attachEvaluationListeners() {
            document.querySelectorAll('.rule-radio').forEach(radio => {
                radio.addEventListener('change', () => {
                    const ruleId = radio.dataset.rule;
                    appState.selectedRules[ruleId] = radio.value === 'yes';
                    render();
                });
            });
            document.getElementById('backEvalBtn').addEventListener('click', () => {
                appState.selection = {};
                appState.selectedRules = {};
                appState.currentScreen = 'AGENT_SELECTION';
                render();
            });
            document.getElementById('submitBtn').addEventListener('click', () => {
                const rules = getRulesByType(appState.selection.type);
                const { score, issues } = calculateScore(appState.selectedRules, rules);
                const newMonitoria = {
                    id: Date.now(),
                    agent: appState.selection.agent,
                    type: appState.selection.type,
                    clientName: appState.selection.clientName,
                    date: appState.selection.date,
                    protocol: appState.selection.protocol,
                    score,
                    issues,
                    selectedRules: appState.selectedRules,
                    timestamp: new Date().toISOString(),
                };
                appState.monitorias.unshift(newMonitoria);
                appState.currentMonitoria = newMonitoria;
                saveState();
                appState.currentScreen = 'RESULT';
                render();
            });
        }
        function attachResultListeners() {
            document.getElementById('newMonitoriaBtn').addEventListener('click', () => {
                appState.selection = {};
                appState.selectedRules = {};
                appState.currentMonitoria = null;
                appState.currentScreen = 'AGENT_SELECTION';
                render();
            });
            document.getElementById('historyBtn').addEventListener('click', () => {
                appState.currentScreen = 'HISTORY';
                render();
            });
        }
        function attachHistoryListeners() {
            document.querySelectorAll('.view-detail').forEach(btn => {
                btn.addEventListener('click', () => {
                    const id = btn.dataset.id;
                    appState.detailMonitoria = appState.monitorias.find(m => m.id == id);
                    appState.currentScreen = 'DETAIL';
                    render();
                });
            });
            document.getElementById('dashboardBtn').addEventListener('click', () => {
                appState.currentScreen = 'DASHBOARD';
                render();
            });
            document.getElementById('newBtn').addEventListener('click', () => {
                appState.currentScreen = 'AGENT_SELECTION';
                appState.selection = {};
                appState.selectedRules = {};
                render();
            });
        }
        function attachDetailListeners() {
            document.getElementById('backDetailBtn').addEventListener('click', () => {
                appState.currentScreen = 'HISTORY';
                render();
            });
        }
        function attachDashboardListeners() {
            document.getElementById('backDashBtn').addEventListener('click', () => {
                appState.currentScreen = 'HISTORY';
                render();
            });
            document.getElementById('newMonFromDashBtn').addEventListener('click', () => {
                appState.currentScreen = 'AGENT_SELECTION';
                appState.selection = {};
                appState.selectedRules = {};
                render();
            });
        }
        // ============================================
        // INIT
        // ============================================
        render();
    </script>
</body>
</html>
