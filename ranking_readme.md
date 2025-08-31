{
    id: 6,
    name: "Universidade Jean Piaget de Angola",
    type: "universidade",
    province: "luanda",
    score: 7.9,
    trend: "-0.1",
    logo: "https://via.placeholder.com/55x55/6f42c1/ffffff?text=UJPA",
    website: "https://www.unipiaget.ao",
    details: {
      founded: 2001,
      students: 6800,
      faculty: 320,
      programs: 28,
      research: 7.2,
      teaching: 8.0,
      reputation: 7.8,
      employability: 8.3,
      infrastructure: 7.5,
      description: "Universidade privada com metodologia europeia e foco na inovação pedagógica.",
      achievements: [
        "Metodologia europeia de ensino",
        "Intercâmbios internacionais",
        "Inovação pedagógica"
      ]
    }
  }
];

// Variáveis globais
let currentPage = 1;
let itemsPerPage = 5;
let filteredInstitutions = [...institutions];
let isLoading = false;

// Inicialização
document.addEventListener('DOMContentLoaded', function() {
    initializeRanking();
});

function initializeRanking() {
    renderRanking(institutions);
    setupFilters();
    setupThemeToggle();
    animateCounters();
    setupPagination();
    setupTableControls();
    
    // Animar barras de pontuação após um pequeno delay
    setTimeout(animateScores, 500);
}

// Renderizar ranking
function renderRanking(data) {
    const tbody = document.getElementById('rankingTableBody');
    if (!tbody) return;
    
    tbody.innerHTML = '';
    
    // Calcular dados para paginação
    const startIndex = (currentPage - 1) * itemsPerPage;
    const endIndex = startIndex + itemsPerPage;
    const paginatedData = data.slice(startIndex, endIndex);
    
    paginatedData.forEach((institution, index) => {
        const globalRank = startIndex + index + 1;
        const row = createInstitutionRow(institution, globalRank);
        tbody.appendChild(row);
    });
    
    updateTableInfo(data.length, startIndex + 1, Math.min(endIndex, data.length));
    updatePaginationControls(data.length);
    
    // Animar entrada das linhas
    setTimeout(() => {
        tbody.querySelectorAll('tr').forEach((row, index) => {
            setTimeout(() => {
                row.style.opacity = '1';
                row.style.transform = 'translateX(0)';
            }, index * 100);
        });
    }, 100);
}

// Criar linha da instituição
function createInstitutionRow(institution, rank) {
    const row = document.createElement('tr');
    row.style.opacity = '0';
    row.style.transform = 'translateX(-20px)';
    row.style.transition = 'all 0.3s ease';
    
    const rankClass = getRankClass(rank);
    const typeClass = `type-${institution.type}`;
    const trendClass = getTrendClass(institution.trend);
    const trendIcon = getTrendIcon(institution.trend);
    
    row.innerHTML = `
        <td>
            <div class="rank-badge ${rankClass}">${rank}</div>
        </td>
        <td>
            <div class="institution-info">
                <img src="${institution.logo}" alt="${institution.name}" class="institution-logo" 
                     onerror="this.src='https://via.placeholder.com/55x55/ff6b35/ffffff?text=${institution.name.charAt(0)}'">
                <div class="institution-details">
                    <h6>${institution.name}</h6>
                    <div class="institution-meta">
                        <i class="fas fa-users"></i>
                        <span>${institution.details.students.toLocaleString('pt-BR')} estudantes</span>
                        <span>•</span>
                        <span>Fundada em ${institution.details.founded}</span>
                    </div>
                </div>
            </div>
        </td>
        <td>
            <span class="type-badge ${typeClass}">
                ${getTypeIcon(institution.type)} ${capitalizeFirst(institution.type)}
            </span>
        </td>
        <td>${capitalizeFirst(institution.province)}</td>
        <td>
            <div class="score-display">
                <div class="score-number">${institution.score}</div>
                <div class="score-bar">
                    <div class="score-fill" data-width="${(institution.score / 10) * 100}"></div>
                </div>
            </div>
        </td>
        <td>
            <span class="trend-badge ${trendClass}">
                ${trendIcon} ${institution.trend}
            </span>
        </td>
        <td>
            <div class="action-buttons">
                <button class="btn btn-action" onclick="showDetails(${institution.id})" title="Ver detalhes">
                    <i class="fas fa-eye"></i>
                </button>
                <button class="btn btn-action" onclick="shareInstitution(${institution.id})" title="Compartilhar">
                    <i class="fas fa-share-alt"></i>
                </button>
            </div>
        </td>
    `;
    
    return row;
}

// Funções auxiliares para classes e ícones
function getRankClass(rank) {
    if (rank === 1) return 'rank-1';
    if (rank === 2) return 'rank-2';
    if (rank === 3) return 'rank-3';
    return 'rank-default';
}

function getTrendClass(trend) {
    if (trend.startsWith('+')) return 'trend-up';
    if (trend.startsWith('-')) return 'trend-down';
    return 'trend-stable';
}

function getTrendIcon(trend) {
    if (trend.startsWith('+')) return '<i class="fas fa-arrow-up"></i>';
    if (trend.startsWith('-')) return '<i class="fas fa-arrow-down"></i>';
    return '<i class="fas fa-minus"></i>';
}

function getTypeIcon(type) {
    const icons = {
        'universidade': '<i class="fas fa-university"></i>',
        'instituto': '<i class="fas fa-building"></i>',
        'centro': '<i class="fas fa-tools"></i>'
    };
    return icons[type] || '<i class="fas fa-school"></i>';
}

function capitalizeFirst(str) {
    return str.charAt(0).toUpperCase() + str.slice(1);
}

// Configurar filtros
function setupFilters() {
    // Filtros de categoria
    document.querySelectorAll('.category-btn').forEach(btn => {
        btn.addEventListener('click', function() {
            if (isLoading) return;
            
            document.querySelectorAll('.category-btn').forEach(b => b.classList.remove('active'));
            this.classList.add('active');
            
            applyFilters();
        });
    });

    // Filtro de província
    const provinciaFilter = document.getElementById('provinciaFilter');
    if (provinciaFilter) {
        provinciaFilter.addEventListener('change', applyFilters);
    }

    // Ordenação
    const sortBy = document.getElementById('sortBy');
    if (sortBy) {
        sortBy.addEventListener('change', applyFilters);
    }
}

// Aplicar filtros
function applyFilters() {
    showLoading(true);
    
    // Obter filtros ativos
    const activeCategory = document.querySelector('.category-btn.active')?.dataset.filter || 'all';
    const selectedProvince = document.getElementById('provinciaFilter')?.value || '';
    const sortCriteria = document.getElementById('sortBy')?.value || 'score';
    
    // Filtrar dados
    let filtered = [...institutions];
    
    // Filtro por categoria
    if (activeCategory !== 'all') {
        filtered = filtered.filter(inst => inst.type === activeCategory);
    }
    
    // Filtro por província
    if (selectedProvince) {
        filtered = filtered.filter(inst => inst.province === selectedProvince);
    }
    
    // Ordenar
    filtered.sort((a, b) => {
        switch(sortCriteria) {
            case 'research':
                return b.details.research - a.details.research;
            case 'teaching':
                return b.details.teaching - a.details.teaching;
            case 'reputation':
                return b.details.reputation - a.details.reputation;
            default:
                return b.score - a.score;
        }
    });
    
    filteredInstitutions = filtered;
    currentPage = 1; // Reset para primeira página
    
    // Simular delay de carregamento
    setTimeout(() => {
        renderRanking(filteredInstitutions);
        showLoading(false);
        
        // Mostrar mensagem se não houver resultados
        if (filtered.length === 0) {
            showNoResults();
        }
    }, 800);
}

// Mostrar loading
function showLoading(show) {
    const overlay = document.getElementById('loadingOverlay');
    if (overlay) {
        overlay.style.display = show ? 'flex' : 'none';
        isLoading = show;
    }
}

// Mostrar quando não há resultados
function showNoResults() {
    const tbody = document.getElementById('rankingTableBody');
    if (!tbody) return;
    
    tbody.innerHTML = `
        <tr>
            <td colspan="7" class="text-center py-5">
                <div>
                    <i class="fas fa-search fa-3x text-muted mb-3"></i>
                    <h5>Nenhuma instituição encontrada</h5>
                    <p class="text-muted">Tente ajustar os filtros para encontrar mais resultados.</p>
                    <button class="btn btn-orange mt-2" onclick="clearFilters()">
                        <i class="fas fa-refresh me-2"></i>Limpar Filtros
                    </button>
                </div>
            </td>
        </tr>
    `;
}

// Limpar filtros
function clearFilters() {
    document.querySelector('.category-btn[data-filter="all"]')?.classList.add('active');
    document.querySelectorAll('.category-btn:not([data-filter="all"])').forEach(btn => {
        btn.classList.remove('active');
    });
    
    const provinciaFilter = document.getElementById('provinciaFilter');
    if (provinciaFilter) provinciaFilter.value = '';
    
    const sortBy = document.getElementById('sortBy');
    if (sortBy) sortBy.value = 'score';
    
    applyFilters();
}

// Configurar paginação
function setupPagination() {
    const prevBtn = document.getElementById('prevPage');
    const nextBtn = document.getElementById('nextPage');
    
    if (prevBtn) {
        prevBtn.addEventListener('click', () => {
            if (currentPage > 1) {
                currentPage--;
                renderRanking(filteredInstitutions);
            }
        });
    }
    
    if (nextBtn) {
        nextBtn.addEventListener('click', () => {
            const totalPages = Math.ceil(filteredInstitutions.length / itemsPerPage);
            if (currentPage < totalPages) {
                currentPage++;
                renderRanking(filteredInstitutions);
            }
        });
    }
}

// Atualizar controles de paginação
function updatePaginationControls(totalItems) {
    const totalPages = Math.ceil(totalItems / itemsPerPage);
    
    const prevBtn = document.getElementById('prevPage');
    const nextBtn = document.getElementById('nextPage');
    const currentPageSpan = document.getElementById('currentPage');
    const totalPagesSpan = document.getElementById('totalPages');
    
    if (prevBtn) prevBtn.disabled = currentPage <= 1;
    if (nextBtn) nextBtn.disabled = currentPage >= totalPages;
    if (currentPageSpan) currentPageSpan.textContent = currentPage;
    if (totalPagesSpan) totalPagesSpan.textContent = totalPages;
}

// Atualizar informações da tabela
function updateTableInfo(total, start, end) {
    const tableInfo = document.getElementById('tableInfo');
    if (tableInfo) {
        tableInfo.textContent = `Mostrando ${start}-${end} de ${total} instituições`;
    }
}

// Configurar controles da tabela
function setupTableControls() {
    const exportBtn = document.getElementById('exportBtn');
    const refreshBtn = document.getElementById('refreshBtn');
    
    if (exportBtn) {
        exportBtn.addEventListener('click', exportRankingData);
    }
    
    if (refreshBtn) {
        refreshBtn.addEventListener('click', refreshRanking);
    }
}

// Exportar dados do ranking
function exportRankingData() {
    const csvContent = generateCSV(filteredInstitutions);
    downloadCSV(csvContent, 'ranking_instituicoes.csv');
    showAlert('Dados exportados com sucesso!', 'success');
}

// Gerar CSV
function generateCSV(data) {
    const headers = ['Posição', 'Instituição', 'Tipo', 'Província', 'Pontuação', 'Tendência', 'Estudantes', 'Fundada'];
    const rows = data.map((inst, index) => [
        index + 1,
        inst.name,
        inst.type,
        inst.province,
        inst.score,
        inst.trend,
        inst.details.students,
        inst.details.founded
    ]);
    
    const csvContent = [headers, ...rows]
        .map(row => row.map(field => `"${field}"`).join(','))
        .join('\n');
    
    return csvContent;
}

// Download CSV
function downloadCSV(content, filename) {
    const blob = new Blob([content], { type: 'text/csv;charset=utf-8;' });
    const link = document.createElement('a');
    link.href = URL.createObjectURL(blob);
    link.download = filename;
    link.click();
}

// Atualizar ranking
function refreshRanking() {
    const refreshBtn = document.getElementById('refreshBtn');
    if (refreshBtn) {
        const icon = refreshBtn.querySelector('i');
        icon.classList.add('fa-spin');
        
        showLoading(true);
        
        setTimeout(() => {
            renderRanking(filteredInstitutions);
            showLoading(false);
            icon.classList.remove('fa-spin');
            showAlert('Ranking atualizado com sucesso!', 'success');
        }, 1500);
    }
}

// Configurar toggle de tema
function setupThemeToggle() {
    const themeToggle = document.getElementById('themeToggle');
    const body = document.body;
    
    if (!themeToggle) return;
    
    // Carregar tema salvo
    const savedTheme = localStorage.getItem('theme') || 'light';
    if (savedTheme === 'dark') {
        body.classList.add('dark-mode');
        updateThemeIcon(true);
    }
    
    themeToggle.addEventListener('click', function() {
        const isDark = body.classList.toggle('dark-mode');
        updateThemeIcon(isDark);
        localStorage.setItem('theme', isDark ? 'dark' : 'light');
        
        showAlert(`Tema ${isDark ? 'escuro' : 'claro'} ativado`, 'info');
    });
}

// Atualizar ícone do tema
function updateThemeIcon(isDark) {
    const icon = document.querySelector('#themeToggle i');
    if (icon) {
        icon.className = isDark ? 'fa-solid fa-sun' : 'fa-solid fa-moon';
    }
}

// Animar contadores
function animateCounters() {
    const counters = document.querySelectorAll('.stat-number[data-count]');
    
    const observer = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                animateCounter(entry.target);
                observer.unobserve(entry.target);
            }
        });
    });
    
    counters.forEach(counter => observer.observe(counter));
}

// Animar contador individual
function animateCounter(element) {
    const target = parseFloat(element.dataset.count);
    const isDecimal = element.dataset.decimal === 'true';
    const increment = target / 100;
    let current = 0;
    
    const timer = setInterval(() => {
        current += increment;
        if (current >= target) {
            current = target;
            clearInterval(timer);
        }
        
        element.textContent = isDecimal ? 
            current.toFixed(1) : 
            Math.floor(current).toLocaleString('pt-BR');
    }, 20);
}

// Animar barras de pontuação
function animateScores() {
    const scoreFills = document.querySelectorAll('.score-fill[data-width]');
    
    const observer = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                const fill = entry.target;
                const width = fill.dataset.width + '%';
                
                setTimeout(() => {
                    fill.style.width = width;
                }, 200);
                
                observer.unobserve(fill);
            }
        });
    });
    
    scoreFills.forEach(fill => observer.observe(fill));
}

// Mostrar detalhes da instituição
function showDetails(institutionId) {
    const institution = institutions.find(inst => inst.id === institutionId);
    if (!institution) return;
    
    const modal = document.getElementById('institutionModal');
    const modalTitle = document.getElementById('modalTitle');
    const modalBody = document.getElementById('modalBody');
    const visitBtn = document.getElementById('visitWebsiteBtn');
    
    if (!modal || !modalTitle || !modalBody) return;
    
    modalTitle.innerHTML = `
        <i class="fas fa-university me-2"></i>
        ${institution.name}
    `;
    
    modalBody.innerHTML = createModalContent(institution);
    
    if (visitBtn) {
        visitBtn.onclick = () => window.open(institution.website, '_blank');
    }
    
    // Mostrar modal
    const bsModal = new bootstrap.Modal(modal);
    bsModal.show();
    
    // Animar gráficos do modal após aparecer
    modal.addEventListener('shown.bs.modal', () => {
        animateModalCharts(institution);
    }, { once: true });
}

// Criar conteúdo do modal
function createModalContent(institution) {
    const details = institution.details;
    
    return `
        <div class="row">
            <div class="col-md-4 text-center mb-4">
                <img src="${institution.logo}" alt="${institution.name}" 
                     class="rounded-circle mb-3" width="100" height="100"
                     style="border: 4px solid var(--primary-orange);"
                     onerror="this.src='https://via.placeholder.com/100x100/ff6b35/ffffff?text=${institution.name.charAt(0)}'">
                <h5 class="text-primary">${institution.name}</h5>
                <p class="text-muted">Fundada em ${details.founded}</p>
                
                <div class="d-grid gap-2 mt-3">
                    <button class="btn btn-outline-orange btn-sm" onclick="shareInstitution(${institution.id})">
                        <i class="fas fa-share-alt me-1"></i> Compartilhar
                    </button>
                </div>
            </div>
            
            <div class="col-md-8">
                <div class="mb-4">
                    <h6 class="text-primary mb-2">
                        <i class="fas fa-info-circle me-2"></i>Sobre a Instituição
                    </h6>
                    <p class="text-muted">${details.description}</p>
                </div>
                
                <div class="row mb-4">
                    <div class="col-sm-6">
                        <div class="d-flex justify-content-between align-items-center mb-2">
                            <span><i class="fas fa-users me-2 text-primary"></i>Estudantes:</span>
                            <strong>${details.students.toLocaleString('pt-BR')}</strong>
                        </div>
                        <div class="d-flex justify-content-between align-items-center mb-2">
                            <span><i class="fas fa-chalkboard-teacher me-2 text-success"></i>Docentes:</span>
                            <strong>${details.faculty.toLocaleString('pt-BR')}</strong>
                        </div>
                    </div>
                    <div class="col-sm-6">
                        <div class="d-flex justify-content-between align-items-center mb-2">
                            <span><i class="fas fa-graduation-cap me-2 text-warning"></i>Programas:</span>
                            <strong>${details.programs}</strong>
                        </div>
                        <div class="d-flex justify-content-between align-items-center mb-2">
                            <span><i class="fas fa-map-marker-alt me-2 text-info"></i>Província:</span>
                            <strong>${capitalizeFirst(institution.province)}</strong>
                        </div>
                    </div>
                </div>
                
                <h6 class="text-primary mb-3">
                    <i class="fas fa-chart-bar me-2"></i>Pontuações Detalhadas
                </h6>
                
                ${createScoreChart('Investigação', details.research, 'primary')}
                ${createScoreChart('Qualidade de Ensino', details.teaching, 'success')}
                ${createScoreChart('Reputação', details.reputation, 'warning')}
                ${createScoreChart('Empregabilidade', details.employability, 'info')}
                ${createScoreChart('Infraestrutura', details.infrastructure, 'secondary')}
            </div>
        </div>
        
        <div class="mt-4">
            <h6 class="text-primary mb-3">
                <i class="fas fa-trophy me-2"></i>Principais Conquistas
            </h6>
            <div class="row">
                ${details.achievements.map(achievement => `
                    <div class="col-md-4 mb-2">
                        <div class="d-flex align-items-start">
                            <i class="fas fa-check-circle text-success me-2 mt-1"></i>
                            <span class="small">${achievement}</span>
                        </div>
                    </div>
                `).join('')}
            </div>
        </div>
    `;
}

// Criar gráfico de pontuação
function createScoreChart(label, score, color) {
    return `
        <div class="mb-3">
            <div class="d-flex justify-content-between align-items-center mb-1">
                <span class="small">${label}</span>
                <span class="fw-bold text-${color}">${score}/10</span>
            </div>
            <div class="progress" style="height: 8px;">
                <div class="progress-bar progress-bar-${color}" 
                     data-score="${score}"
                     style="width: 0%"
                     role="progressbar">
                </div>
            </div>
        </div>
    `;
}

// Animar gráficos do modal
function animateModalCharts(institution) {
    const progressBars = document.querySelectorAll('.progress-bar[data-score]');
    
    progressBars.forEach((bar, index) => {
        setTimeout(() => {
            const score = parseFloat(bar.dataset.score);
            bar.style.width = (score / 10 * 100) + '%';
        }, index * 200);
    });
}

// Compartilhar instituição
function shareInstitution(institutionId) {
    const institution = institutions.find(inst => inst.id === institutionId);
    if (!institution) return;
    
    const shareData = {
        title: `${institution.name} - Ranking Academicoapp`,
        text: `Conheça a ${institution.name}, pontuação ${institution.score}/10 no ranking de instituições acadêmicas de Angola.`,
        url: `${window.location.href}?institution=${institutionId}`
    };
    
    if (navigator.share) {
        navigator.share(shareData)
            .then(() => showAlert('Link compartilhado com sucesso!', 'success'))
            .catch(() => fallbackShare(shareData.url));
    } else {
        fallbackShare(shareData.url);
    }
}

// Compartilhamento alternativo
function fallbackShare(url) {
    if (navigator.clipboard) {
        navigator.clipboard.writeText(url)
            .then(() => showAlert('Link copiado para área de transferência!', 'success'))
            .catch(() => showManualShareAlert(url));
    } else {
        showManualShareAlert(url);
    }
}

// Mostrar alerta de compartilhamento manual
function showManualShareAlert(url) {
    showAlert('Copie o link: ' + url, 'info', 5000);
}

// Sistema de alertas
function showAlert(message, type = 'info', duration = 3000) {
    // Remove alertas anteriores
    const existingAlert = document.querySelector('.alert-notification');
    if (existingAlert) {
        existingAlert.remove();
    }
    
    // Criar alerta
    const alert = document.createElement('div');
    alert.className = `alert-notification alert-${type}`;
    
    const icons = {
        success: 'fas fa-check-circle',
        error: 'fas fa-exclamation-circle',
        warning: 'fas fa-exclamation-triangle',
        info: 'fas fa-info-circle'
    };
    
    alert.innerHTML = `
        <div class="alert-content">
            <i class="${icons[type] || icons.info} me-2"></i>
            <span>${message}</span>
            <button class="alert-close" onclick="this.parentElement.parentElement.remove()">
                <i class="fas fa-times"></i>
            </button>
        </div>
    `;
    
    // Estilos inline
    alert.style.cssText = `
        position: fixed;
        top: 20px;
        right: 20px;
        z-index: 9999;
        max-width: 400px;
        padding: 16px 20px;
        border-radius: 12px;
        box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
        animation: slideInRight 0.3s ease;
        font-size: 14px;
        font-weight: 500;
        ${getAlertStyles(type)}
    `;
    
    document.body.appendChild(alert);
    
    // Auto remover
    setTimeout(() => {
        if (alert.parentElement) {
            alert.style.animation = 'slideOutRight 0.3s ease forwards';
            setTimeout(() => alert.remove(), 300);
        }
    }, duration);
}

// Estilos dos alertas
function getAlertStyles(type) {
    const styles = {
        success: 'background: #d4edda; border: 1px solid #c3e6cb; color: #155724;',
        error: 'background: #f8d7da; border: 1px solid #f5c6cb; color: #721c24;',
        warning: 'background: #fff3cd; border: 1px solid #ffeaa7; color: #856404;',
        info: 'background: #d1ecf1; border: 1px solid #bee5eb; color: #0c5460;'
    };
    return styles[type] || styles.info;
}

// Adicionar estilos das animações
function addAnimationStyles() {
    const style = document.createElement('style');
    style.textContent = `
        .alert-notification .alert-content {
            display: flex;
            align-items: center;
            gap: 8px;
        }
        
        .alert-notification .alert-close {
            background: none;
            border: none;
            cursor: pointer;
            padding: 4px;
            margin-left: auto;
            opacity: 0.7;
            transition: opacity 0.2s;
        }
        
        .alert-notification .alert-close:hover {
            opacity: 1;
        }
        
        @keyframes slideInRight {
            from { transform: translateX(100%); opacity: 0; }
            to { transform: translateX(0); opacity: 1; }
        }
        
        @keyframes slideOutRight {
            from { transform: translateX(0); opacity: 1; }
            to { transform: translateX(100%); opacity: 0; }
        }
        
        .progress-bar-primary { background: var(--primary-orange) !important; }
        .progress-bar-success { background: var(--success-color) !important; }
        .progress-bar-warning { background: var(--warning-color) !important; }
        .progress-bar-info { background: var(--info-color) !important; }
        .progress-bar-secondary { background: var(--medium-gray) !important; }
    `;
    document.head.appendChild(style);
}

// Funcionalidades de debug (remover em produção)
function debugRanking() {
    console.log({
        institutions: institutions,
        filtered: filteredInstitutions,
        currentPage: currentPage,
        itemsPerPage: itemsPerPage,
        isLoading: isLoading
    });
}

// Funções de utilidade
function generateRandomScore() {
    return Math.round((Math.random() * 2 + 7) * 10) / 10;
}

function shuffleArray(array) {
    const shuffled = [...array];
    for (let i = shuffled.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
    }
    return shuffled;
}

// Inicializar estilos das animações
document.addEventListener('DOMContentLoaded', addAnimationStyles);

// Tratamento de erros global
window.addEventListener('error', function(e) {
    console.error('Erro na aplicação:', e.error);
    showAlert('Ocorreu um erro inesperado. Tente recarregar a página.', 'error', 5000);
});

// Funcionalidade de pesquisa por URL
function handleURLParams() {
    const urlParams = new URLSearchParams(window.location.search);
    const institutionId = urlParams.get('institution');
    
    if (institutionId) {
        // Aguardar carregamento completo antes de mostrar detalhes
        setTimeout(() => {
            showDetails(parseInt(institutionId));
        }, 1000);
    }
}

// Pesquisa rápida (funcionalidade futura)
function setupQuickSearch() {
    const searchInput = document.getElementById('quickSearch');
    if (!searchInput) return;
    
    let searchTimeout;
    searchInput.addEventListener('input', function() {
        clearTimeout(searchTimeout);
        searchTimeout = setTimeout(() => {
            const query = this.value.toLowerCase().trim();
            
            if (query.length === 0) {
                filteredInstitutions = [...institutions];
            } else {
                filteredInstitutions = institutions.filter(inst =>
                    inst.name.toLowerCase().includes(query) ||
                    inst.province.toLowerCase().includes(query) ||
                    inst.type.toLowerCase().includes(query)
                );
            }
            
            currentPage = 1;
            renderRanking(filteredInstitutions);
        }, 300);
    });
}

// Comparar instituições
function compareInstitutions(id1, id2) {
    const inst1 = institutions.find(i => i.id === id1);
    const inst2 = institutions.find(i => i.id === id2);
    
    if (!inst1 || !inst2) return;
    
    const comparison = {
        institution1: inst1,
        institution2: inst2,
        differences: {
            score: Math.abs(inst1.score - inst2.score),
            research: Math.abs(inst1.details.research - inst2.details.research),
            teaching: Math.abs(inst1.details.teaching - inst2.details.teaching),
            reputation: Math.abs(inst1.details.reputation - inst2.details.reputation),
            employability: Math.abs(inst1.details.employability - inst2.details.employability),
            infrastructure: Math.abs(inst1.details.infrastructure - inst2.details.infrastructure)
        }
    };
    
    return comparison;
}

// Favoritar instituição
function toggleFavorite(institutionId) {
    let favorites = JSON.parse(localStorage.getItem('favoriteInstitutions') || '[]');
    
    if (favorites.includes(institutionId)) {
        favorites = favorites.filter(id => id !== institutionId);
        showAlert('Instituição removida dos favoritos', 'info');
    } else {
        favorites.push(institutionId);
        showAlert('Instituição adicionada aos favoritos', 'success');
    }
    
    localStorage.setItem('favoriteInstitutions', JSON.stringify(favorites));
    updateFavoriteButtons();
}

// Atualizar botões de favorito
function updateFavoriteButtons() {
    const favorites = JSON.parse(localStorage.getItem('favoriteInstitutions') || '[]');
    
    document.querySelectorAll('.btn-favorite').forEach(btn => {
        const institutionId = parseInt(btn.dataset.institutionId);
        const isFavorite = favorites.includes(institutionId);
        
        btn.classList.toggle('active', isFavorite);
        const icon = btn.querySelector('i');
        if (icon) {
            icon.className = isFavorite ? 'fas fa-heart' : 'far fa-heart';
        }
    });
}

// Filtrar por favoritos
function showOnlyFavorites() {
    const favorites = JSON.parse(localStorage.getItem('favoriteInstitutions') || '[]');
    
    if (favorites.length === 0) {
        showAlert('Você não tem instituições favoritas ainda', 'warning');
        return;
    }
    
    filteredInstitutions = institutions.filter(inst => favorites.includes(inst.id));
    currentPage = 1;
    renderRanking(filteredInstitutions);
    
    showAlert(`Mostrando ${favorites.length} instituições favoritas`, 'info');
}

// Analytics de interação
function trackInteraction(action, data = {}) {
    const interaction = {
        action: action,
        timestamp: new Date().toISOString(),
        userAgent: navigator.userAgent,
        ...data
    };
    
    // Em produção, enviar para serviço de analytics
    console.log('Interaction tracked:', interaction);
    
    // Salvar localmente para debug
    const interactions = JSON.parse(localStorage.getItem('userInteractions') || '[]');
    interactions.push(interaction);
    
    // Manter apenas as últimas 100 interações
    if (interactions.length > 100) {
        interactions.splice(0, interactions.length - 100);
    }
    
    localStorage.setItem('userInteractions', JSON.stringify(interactions));
}

// Relatório de analytics
function generateAnalyticsReport() {
    const interactions = JSON.parse(localStorage.getItem('userInteractions') || '[]');
    
    const report = {
        totalInteractions: interactions.length,
        uniqueActions: [...new Set(interactions.map(i => i.action))],
        mostViewedInstitutions: getMostViewedInstitutions(interactions),
        popularFilters: getPopularFilters(interactions),
        sessionDuration: calculateSessionDuration(interactions),
        deviceInfo: getDeviceInfo()
    };
    
    return report;
}

// Obter instituições mais visualizadas
function getMostViewedInstitutions(interactions) {
    const views = interactions
        .filter(i => i.action === 'view_details')
        .reduce((acc, i) => {
            acc[i.institutionId] = (acc[i.institutionId] || 0) + 1;
            return acc;
        }, {});
    
    return Object.entries(views)
        .sort(([,a], [,b]) => b - a)
        .slice(0, 5)
        .map(([id, count]) => ({
            institution: institutions.find(i => i.id === parseInt(id))?.name || 'Unknown',
            views: count
        }));
}

// Obter filtros mais populares
function getPopularFilters(interactions) {
    return interactions
        .filter(i => i.action === 'apply_filter')
        .reduce((acc, i) => {
            const key = `${i.filterType}:${i.filterValue}`;
            acc[key] = (acc[key] || 0) + 1;
            return acc;
        }, {});
}

// Calcular duração da sessão
function calculateSessionDuration(interactions) {
    if (interactions.length < 2) return 0;
    
    const first = new Date(interactions[0].timestamp);
    const last = new Date(interactions[interactions.length - 1].timestamp);
    
    return Math.round((last - first) / 1000); // em segundos
}

// Obter informações do dispositivo
function getDeviceInfo() {
    return {
        screenWidth: window.screen.width,
        screenHeight: window.screen.height,
        windowWidth: window.innerWidth,
        windowHeight: window.innerHeight,
        isMobile: window.innerWidth <= 768,
        isTablet: window.innerWidth > 768 && window.innerWidth <= 1024,
        language: navigator.language,
        platform: navigator.platform,
        cookieEnabled: navigator.cookieEnabled
    };
}

// Configurar Service Worker (para funcionalidade offline)
function setupServiceWorker() {
    if ('serviceWorker' in navigator) {
        navigator.serviceWorker.register('/sw.js')
            .then(registration => {
                console.log('SW registered:', registration);
            })
            .catch(error => {
                console.log('SW registration failed:', error);
            });
    }
}

// Cache de dados
class RankingCache {
    constructor() {
        this.cacheName = 'ranking-cache';
        this.maxAge = 5 * 60 * 1000; // 5 minutos
    }
    
    set(key, data) {
        const cacheData = {
            data: data,
            timestamp: Date.now()
        };
        localStorage.setItem(`${this.cacheName}-${key}`, JSON.stringify(cacheData));
    }
    
    get(key) {
        const cached = localStorage.getItem(`${this.cacheName}-${key}`);
        if (!cached) return null;
        
        const { data, timestamp } = JSON.parse(cached);
        
        if (Date.now() - timestamp > this.maxAge) {
            this.delete(key);
            return null;
        }
        
        return data;
    }
    
    delete(key) {
        localStorage.removeItem(`${this.cacheName}-${key}`);
    }
    
    clear() {
        Object.keys(localStorage)
            .filter(key => key.startsWith(this.cacheName))
            .forEach(key => localStorage.removeItem(key));
    }
}

// Instância do cache
const rankingCache = new RankingCache();

// Salvar estado da página
function savePageState() {
    const state = {
        currentPage: currentPage,
        activeFilters: {
            category: document.querySelector('.category-btn.active')?.dataset.filter,
            province: document.getElementById('provinciaFilter')?.value,
            sort: document.getElementById('sortBy')?.value
        },
        timestamp: Date.now()
    };
    
    rankingCache.set('pageState', state);
}

// Restaurar estado da página
function restorePageState() {
    const state = rankingCache.get('pageState');
    if (!state) return;
    
    // Restaurar filtros
    if (state.activeFilters.category && state.activeFilters.category !== 'all') {
        const categoryBtn = document.querySelector(`[data-filter="${state.activeFilters.category}"]`);
        if (categoryBtn) {
            document.querySelectorAll('.category-btn').forEach(btn => btn.classList.remove('active'));
            categoryBtn.classList.add('active');
        }
    }
    
    if (state.activeFilters.province) {
        const provinciaFilter = document.getElementById('provinciaFilter');
        if (provinciaFilter) provinciaFilter.value = state.activeFilters.province;
    }
    
    if (state.activeFilters.sort) {
        const sortBy = document.getElementById('sortBy');
        if (sortBy) sortBy.value = state.activeFilters.sort;
    }
    
    // Aplicar filtros restaurados
    applyFilters();
    
    // Restaurar página
    currentPage = state.currentPage || 1;
}

// Configurações de acessibilidade
function setupAccessibility() {
    // Atalhos de teclado
    document.addEventListener('keydown', function(e) {
        // Alt + F: Focar no primeiro filtro
        if (e.altKey && e.key === 'f') {
            e.preventDefault();
            document.querySelector('.category-btn')?.focus();
        }
        
        // Alt + T: Focar na tabela
        if (e.altKey && e.key === 't') {
            e.preventDefault();
            document.querySelector('#rankingTableBody tr')?.focus();
        }
        
        // Alt + S: Pesquisa rápida
        if (e.altKey && e.key === 's') {
            e.preventDefault();
            document.getElementById('quickSearch')?.focus();
        }
        
        // Esc: Fechar modal
        if (e.key === 'Escape') {
            const modal = document.querySelector('.modal.show');
            if (modal) {
                bootstrap.Modal.getInstance(modal)?.hide();
            }
        }
    });
    
    // Anúncios para screen readers
    const announcer = document.createElement('div');
    announcer.setAttribute('aria-live', 'polite');
    announcer.setAttribute('aria-atomic', 'true');
    announcer.className = 'sr-only';
    document.body.appendChild(announcer);
    
    // Função para anunciar mudanças
    window.announceToScreenReader = function(message) {
        announcer.textContent = message;
        setTimeout(() => announcer.textContent = '', 1000);
    };
}

// Melhorias de performance
function setupPerformanceOptimizations() {
    // Lazy loading de imagens
    const images = document.querySelectorAll('img[data-src]');
    const imageObserver = new IntersectionObserver((entries, observer) => {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                const img = entry.target;
                img.src = img.dataset.src;
                img.removeAttribute('data-src');
                observer.unobserve(img);
            }
        });
    });
    
    images.forEach(img => imageObserver.observe(img));
    
    // Debounce para redimensionamento
    let resizeTimeout;
    window.addEventListener('resize', function() {
        clearTimeout(resizeTimeout);
        resizeTimeout = setTimeout(() => {
            // Reajustar layout se necessário
            if (window.innerWidth <= 768 && !document.body.classList.contains('mobile-view')) {
                document.body.classList.add('mobile-view');
            } else if (window.innerWidth > 768 && document.body.classList.contains('mobile-view')) {
                document.body.classList.remove('mobile-view');
            }
        }, 150);
    });
}

// Inicialização completa
function initializeApplication() {
    // Verificar suporte do navegador
    if (!window.fetch) {
        showAlert('Seu navegador não é suportado. Por favor, atualize para uma versão mais recente.', 'error');
        return;
    }
    
    // Configurar funcionalidades
    setupServiceWorker();
    setupAccessibility();
    setupPerformanceOptimizations();
    setupQuickSearch();
    
    // Restaurar estado anterior
    restorePageState();
    
    // Processar parâmetros da URL
    handleURLParams();
    
    // Salvar interação inicial
    trackInteraction('page_load', {
        referrer: document.referrer,
        timestamp: new Date().toISOString()
    });
    
    console.log('Ranking Academicoapp inicializado com sucesso!');
}

// Executar inicialização completa após DOM carregar
document.addEventListener('DOMContentLoaded', function() {
    initializeRanking();
    setTimeout(initializeApplication, 100);
});

// Salvar estado antes de sair da página
window.addEventListener('beforeunload', function() {
    savePageState();
    trackInteraction('page_unload');
});

// Funcionalidades experimentais (desenvolvimento)
if (typeof window !== 'undefined' && window.location.hostname === 'localhost') {
    window.debugFunctions = {
        generateReport: generateAnalyticsReport,
        clearCache: () => rankingCache.clear(),
        simulateData: () => {
            // Adicionar dados simulados para teste
            const mockInstitutions = Array.from({length: 20}, (_, i) => ({
                id: institutions.length + i + 1,
                name: `Instituição Teste ${i + 1}`,
                type: ['universidade', 'instituto', 'centro'][i % 3],
                province: ['luanda', 'benguela', 'huila', 'bie'][i % 4],
                score: generateRandomScore(),
                trend: ['+0.1', '+0.2', '-0.1', '0.0'][i % 4],
                logo: `https://via.placeholder.com/55x55/ff6b35/ffffff?text=T${i+1}`,
                website: `https://test${i+1}.edu.ao`,
                details: {
                    founded: 2000 + (i % 24),
                    students: Math.floor(Math.random() * 40000) + 1000,
                    faculty: Math.floor(Math.random() * 2000) + 100,
                    programs: Math.floor(Math.random() * 100) + 10,
                    research: generateRandomScore(),
                    teaching: generateRandomScore(),
                    reputation: generateRandomScore(),
                    employability: generateRandomScore(),
                    infrastructure: generateRandomScore(),
                    description: 'Instituição de teste para desenvolvimento.',
                    achievements: ['Teste 1', 'Teste 2', 'Teste 3']
                }
            }));
            
            institutions.push(...mockInstitutions);
            filteredInstitutions = [...institutions];
            renderRanking(filteredInstitutions);
            
            console.log(`Adicionadas ${mockInstitutions.length} instituições de teste`);
        }
    };
}// Ranking de Instituições - Academicoapp
// Dados das instituições (simulados)
const institutions = [
  {
    id: 1,
    name: "Universidade Agostinho Neto",
    type: "universidade",
    province: "luanda",
    score: 9.2,
    trend: "+0.3",
    logo: "https://via.placeholder.com/55x55/ff6b35/ffffff?text=UAN",
    website: "https://www.uan.ao",
    details: {
      founded: 1962,
      students: 45000,
      faculty: 2500,
      programs: 127,
      research: 8.9,
      teaching: 9.1,
      reputation: 9.5,
      employability: 8.8,
      infrastructure: 9.0,
      description: "A maior e mais antiga universidade de Angola, reconhecida pela excelência acadêmica e contribuição para o desenvolvimento nacional.",
      achievements: [
        "Primeira universidade de Angola",
        "Mais de 127 cursos oferecidos",
        "Centro de excelência em investigação"
      ]
    }
  },
  {
    id: 2,
    name: "Universidade Católica de Angola",
    type: "universidade", 
    province: "luanda",
    score: 8.8,
    trend: "+0.2",
    logo: "https://via.placeholder.com/55x55/28a745/ffffff?text=UCAN",
    website: "https://www.ucan.edu",
    details: {
      founded: 1997,
      students: 12000,
      faculty: 800,
      programs: 45,
      research: 8.5,
      teaching: 9.0,
      reputation: 8.9,
      employability: 9.1,
      infrastructure: 8.7,
      description: "Universidade privada de referência, conhecida pela qualidade de ensino e valores cristãos.",
      achievements: [
        "Excelência em formação humanística",
        "Parcerias internacionais sólidas",
        "Alta taxa de empregabilidade"
      ]
    }
  },
  {
    id: 3,
    name: "Universidade Mandume ya Ndemufayo",
    type: "universidade",
    province: "huila",
    score: 8.5,
    trend: "+0.5",
    logo: "https://via.placeholder.com/55x55/17a2b8/ffffff?text=UMN",
    website: "https://www.umn.ao",
    details: {
      founded: 2009,
      students: 8500,
      faculty: 450,
      programs: 32,
      research: 7.8,
      teaching: 8.7,
      reputation: 8.2,
      employability: 8.9,
      infrastructure: 8.1,
      description: "Universidade pública com forte crescimento e impacto no desenvolvimento da região sul.",
      achievements: [
        "Crescimento consistente",
        "Foco no desenvolvimento regional",
        "Programas inovadores"
      ]
    }
  },
  {
    id: 4,
    name: "Instituto Superior Politécnico de Tecnologias",
    type: "instituto",
    province: "luanda",
    score: 8.3,
    trend: "0.0",
    logo: "https://via.placeholder.com/55x55/ffc107/000000?text=ISPT",
    website: "https://www.ispt.ao",
    details: {
      founded: 2005,
      students: 3200,
      faculty: 180,
      programs: 15,
      research: 7.5,
      teaching: 8.5,
      reputation: 8.0,
      employability: 9.2,
      infrastructure: 8.8,
      description: "Instituto especializado em tecnologia com forte ligação ao mercado de trabalho.",
      achievements: [
        "Alta empregabilidade dos graduados",
        "Infraestrutura tecnológica avançada",
        "Parcerias com empresas tech"
      ]
    }
  },
  {
    id: 5,
    name: "Centro de Formação Profissional do Lobito",
    type: "centro",
    province: "benguela",
    score: 8.0,
    trend: "+0.1",
    logo: "https://via.placeholder.com/55x55/dc3545/ffffff?text=CFPL",
    website: "https://www.cfpl.ao",
    details: {
      founded: 2010,
      students: 1500,
      faculty: 95,
      programs: 12,
      research: 6.8,
      teaching: 8.2,
      reputation: 7.5,
      employability: 9.5,
      infrastructure: 7.8,
      description: "Centro focado na formação profissional prática com excelente inserção no mercado.",
      achievements: [
        "95% de empregabilidade",
        "Formação prática especializada",
        "Parcerias industriais"
      ]
    }
  },
  {
    id: 6,
    name: "Universidade Jean Piaget de Angola",
    type: "universidade",
    province: "luanda",
    score: 7.9,
    trend: "-0.1",
    logo: "https://via.placeholder.com/55x55