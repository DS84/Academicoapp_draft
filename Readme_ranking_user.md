<div class="podium-achievements">
                        ${user.achievements.slice(0, 3).map(achievement => `
                            <span class="achievement-badge ${achievement}" 
                                  title="${achievementData[achievement].name}">
                            </span>
                        `).join('')}
                    </div>
                </div>
            `;
        }
    });
    
    podium.innerHTML = podiumHtml;
}

// Renderizar ranking
function renderRanking() {
    const tbody = document.getElementById('rankingTableBody');
    if (!tbody) return;
    
    const users = userData[currentUserType] || [];
    filteredUsers = [...users].sort((a, b) => b.points - a.points);
    
    // Aplicar filtros
    applyCurrentFilters();
    
    tbody.innerHTML = '';
    
    const startIndex = (currentPage - 1) * itemsPerPage;
    const endIndex = startIndex + itemsPerPage;
    const pageUsers = filteredUsers.slice(startIndex, endIndex);
    
    pageUsers.forEach((user, index) => {
        const globalRank = startIndex + index + 1;
        const row = createUserRow(user, globalRank);
        tbody.appendChild(row);
    });
    
    updateTableInfo();
    updatePaginationControls();
    
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

// Criar linha do usuário
function createUserRow(user, rank) {
    const row = document.createElement('tr');
    row.style.opacity = '0';
    row.style.transform = 'translateX(-20px)';
    row.style.transition = 'all 0.3s ease';
    
    const rankClass = getRankClass(rank);
    const levelProgress = (user.level / 10) * 100;
    const activityClass = `activity-${user.activity}`;
    const activityText = getActivityText(user.activity);
    
    row.innerHTML = `
        <td>
            <div class="rank-badge ${rankClass}">${rank}</div>
        </td>
        <td>
            <div class="user-info">
                <img src="${user.avatar}" alt="${user.name}" class="user-avatar"
                     onerror="this.src='https://via.placeholder.com/55x55/ff6b35/ffffff?text=${user.name.charAt(0)}'">
                <div class="user-details">
                    <h6>${user.name}</h6>
                    <div class="user-meta">
                        <i class="fas fa-graduation-cap"></i>
                        <span>${getCourseDisplayName(user.course)}</span>
                        <span>•</span>
                        <span>${getInstitutionDisplayName(user.institution)}</span>
                    </div>
                </div>
            </div>
        </td>
        <td>
            <div class="level-indicator">
                <div class="level-number">${user.level}</div>
                <div class="level-bar">
                    <div class="level-fill ${currentUserType}" style="width: ${levelProgress}%"></div>
                </div>
            </div>
        </td>
        <td>
            <div class="points-display">
                <div class="points-number">${user.points.toLocaleString('pt-BR')}</div>
                <div class="points-label">pontos</div>
            </div>
        </td>
        <td>
            <div class="achievement-badges">
                ${user.achievements.map(achievement => `
                    <span class="achievement-badge ${achievement}" 
                          onclick="showAchievementDetails('${achievement}')"
                          title="${achievementData[achievement].name}">
                    </span>
                `).join('')}
            </div>
        </td>
        <td>
            <div class="activity-status">
                <div class="activity-indicator ${activityClass}"></div>
                <span>${activityText}</span>
            </div>
        </td>
        <td>
            <div class="action-buttons">
                <button class="btn btn-action" onclick="showUserProfile(${user.id})" title="Ver perfil">
                    <i class="fas fa-eye"></i>
                </button>
                <button class="btn btn-action" onclick="sendMessage(${user.id})" title="Enviar mensagem">
                    <i class="fas fa-envelope"></i>
                </button>
            </div>
        </td>
    `;
    
    return row;
}

// Funções auxiliares
function getRankClass(rank) {
    if (rank === 1) return 'rank-1';
    if (rank === 2) return 'rank-2';
    if (rank === 3) return 'rank-3';
    return 'rank-default';
}

function getActivityText(activity) {
    const activities = {
        online: 'Online',
        recent: 'Recente',
        offline: 'Offline'
    };
    return activities[activity] || 'Desconhecido';
}

function getCourseDisplayName(course) {
    const courses = {
        informatica: 'Eng. Informática',
        gestao: 'Gestão',
        direito: 'Direito',
        medicina: 'Medicina',
        economia: 'Economia'
    };
    return courses[course] || course;
}

function getInstitutionDisplayName(institution) {
    const institutions = {
        uan: 'UAN',
        ucan: 'UCAN',
        isc: 'ISC'
    };
    return institutions[institution] || institution;
}

// Mostrar perfil do usuário
function showUserProfile(userId) {
    // Buscar usuário em todos os tipos
    let user = null;
    for (const type in userData) {
        user = userData[type].find(u => u.id === userId);
        if (user) break;
    }
    
    if (!user) return;
    
    const modal = document.getElementById('userModal');
    const modalTitle = document.getElementById('userModalTitle');
    const modalBody = document.getElementById('userModalBody');
    
    if (!modal || !modalTitle || !modalBody) return;
    
    modalTitle.innerHTML = `
        <i class="fas fa-user me-2"></i>
        Perfil de ${user.name}
    `;
    
    modalBody.innerHTML = createUserProfileContent(user);
    
    const bsModal = new bootstrap.Modal(modal);
    bsModal.show();
}

// Criar conteúdo do perfil do usuário
function createUserProfileContent(user) {
    return `
        <div class="row">
            <div class="col-md-4 text-center mb-4">
                <img src="${user.avatar}" alt="${user.name}" 
                     class="rounded-circle mb-3" width="120" height="120"
                     style="border: 4px solid var(--primary-orange);"
                     onerror="this.src='https://via.placeholder.com/120x120/ff6b35/ffffff?text=${user.name.charAt(0)}'">
                <h5>${user.name}</h5>
                <p class="text-muted">Nível ${user.level} • ${user.points.toLocaleString('pt-BR')} pontos</p>
                <div class="activity-badge ${user.activity}">
                    <i class="fas fa-circle me-1"></i>
                    ${getActivityText(user.activity)}
                </div>
            </div>
            
            <div class="col-md-8">
                <div class="mb-4">
                    <h6 class="text-primary">
                        <i class="fas fa-info-circle me-2"></i>Informações Gerais
                    </h6>
                    <div class="info-grid">
                        <div class="info-item">
                            <span class="info-label">Curso:</span>
                            <span class="info-value">${getCourseDisplayName(user.course)}</span>
                        </div>
                        <div class="info-item">
                            <span class="info-label">Instituição:</span>
                            <span class="info-value">${getInstitutionDisplayName(user.institution)}</span>
                        </div>
                        <div class="info-item">
                            <span class="info-label">Membro desde:</span>
                            <span class="info-value">${user.details.joinDate}</span>
                        </div>
                    </div>
                </div>
                
                <div class="mb-4">
                    <h6 class="text-primary">
                        <i class="fas fa-trophy me-2"></i>Conquistas
                    </h6>
                    <div class="d-flex flex-wrap gap-2 mb-3">
                        ${user.achievements.map(achievement => `
                            <div class="achievement-badge ${achievement} large" 
                                 onclick="showAchievementDetails('${achievement}')">
                                <i class="${achievementData[achievement].icon}"></i>
                                <span class="ms-2">${achievementData[achievement].name}</span>
                            </div>
                        `).join('')}
                    </div>
                </div>
                
                ${createUserStats(user)}
            </div>
        </div>
        
        <style>
            .activity-badge {
                display: inline-flex;
                align-items: center;
                padding: 6px 12px;
                border-radius: 15px;
                font-size: 12px;
                font-weight: 600;
            }
            .activity-badge.online { background: rgba(40, 167, 69, 0.1); color: var(--success-color); }
            .activity-badge.recent { background: rgba(245, 158, 11, 0.1); color: var(--warning-color); }
            .activity-badge.offline { background: rgba(108, 117, 125, 0.1); color: var(--medium-gray); }
            
            .info-grid {
                display: grid;
                gap: 12px;
            }
            
            .info-item {
                display: flex;
                justify-content: space-between;
                align-items: center;
                padding: 8px 0;
                border-bottom: 1px solid var(--light-gray);
            }
            
            .info-label {
                font-weight: 600;
                color: var(--dark-gray);
            }
            
            .info-value {
                color: var(--medium-gray);
            }
            
            .achievement-badge.large {
                padding: 8px 12px;
                border-radius: 20px;
                font-size: 12px;
                cursor: pointer;
                transition: transform 0.2s ease;
                display: flex;
                align-items: center;
            }
            
            .achievement-badge.large:hover {
                transform: scale(1.05);
            }
        </style>
    `;
}

// Criar estatísticas do usuário baseado no tipo
function createUserStats(user) {
    // Determinar tipo do usuário
    let userType = 'students';
    if (userData.teachers.some(t => t.id === user.id)) userType = 'teachers';
    if (userData.mentors.some(m => m.id === user.id)) userType = 'mentors';
    
    if (userType === 'students') {
        return `
            <div class="mb-4">
                <h6 class="text-primary">
                    <i class="fas fa-chart-bar me-2"></i>Estatísticas Acadêmicas
                </h6>
                <div class="stats-list">
                    <div class="stat-row">
                        <span>Cursos Completados:</span>
                        <strong>${user.details.completedCourses}</strong>
                    </div>
                    <div class="stat-row">
                        <span>Média Geral:</span>
                        <strong>${user.details.averageGrade}/20</strong>
                    </div>
                    <div class="stat-row">
                        <span>Contribuições:</span>
                        <strong>${user.details.contributions}</strong>
                    </div>
                    <div class="stat-row">
                        <span>Estudantes Ajudados:</span>
                        <strong>${user.details.helpedStudents}</strong>
                    </div>
                </div>
            </div>
        `;
    } else if (userType === 'teachers') {
        return `
            <div class="mb-4">
                <h6 class="text-primary">
                    <i class="fas fa-chart-bar me-2"></i>Estatísticas de Ensino
                </h6>
                <div class="stats-list">
                    <div class="stat-row">
                        <span>Cursos Lecionando:</span>
                        <strong>${user.details.coursesTeaching}</strong>
                    </div>
                    <div class="stat-row">
                        <span>Estudantes Orientados:</span>
                        <strong>${user.details.studentsHelped}</strong>
                    </div>
                    <div class="stat-row">
                        <span>Publicações:</span>
                        <strong>${user.details.publications}</strong>
                    </div>
                    <div class="stat-row">
                        <span>Avaliação:</span>
                        <strong>${user.details.rating}/5.0 ⭐</strong>
                    </div>
                </div>
            </div>
        `;
    } else if (userType === 'mentors') {
        return `
            <div class="mb-4">
                <h6 class="text-primary">
                    <i class="fas fa-chart-bar me-2"></i>Estatísticas de Mentoria
                </h6>
                <div class="stats-list">
                    <div class="stat-row">
                        <span>Estudantes Mentoreados:</span>
                        <strong>${user.details.mentoredStudents}</strong>
                    </div>
                    <div class="stat-row">
                        <span>Taxa de Sucesso:</span>
                        <strong>${user.details.successRate}%</strong>
                    </div>
                    <div class="stat-row">
                        <span>Sessões Completadas:</span>
                        <strong>${user.details.sessionsCompleted}</strong>
                    </div>
                    <div class="stat-row">
                        <span>Especialização:</span>
                        <strong>${user.details.specialization}</strong>
                    </div>
                </div>
            </div>
        `;
    }
    
    return '';
}

// Mostrar detalhes da conquista
function showAchievementDetails(achievementKey) {
    const achievement = achievementData[achievementKey];
    if (!achievement) return;
    
    const modal = document.getElementById('achievementModal');
    const modalBody = document.getElementById('achievementModalBody');
    
    if (!modal || !modalBody) return;
    
    modalBody.innerHTML = `
        <div class="text-center mb-4">
            <div class="achievement-icon-large" style="background-color: ${achievement.color}">
                <i class="${achievement.icon}"></i>
            </div>
            <h4 class="mt-3 mb-2">${achievement.name}</h4>
            <p class="text-muted">${achievement.description}</p>
        </div>
        
        <div class="achievement-criteria">
            <h6 class="mb-3">Como obter esta conquista:</h6>
            ${getAchievementCriteria(achievementKey)}
        </div>
        
        <style>
            .achievement-icon-large {
                width: 80px;
                height: 80px;
                border-radius: 50%;
                display: flex;
                align-items: center;
                justify-content: center;
                color: white;
                font-size: 32px;
                margin: 0 auto;
                box-shadow: 0 8px 25px rgba(0, 0, 0, 0.2);
            }
            
            .achievement-criteria ul {
                list-style: none;
                padding: 0;
            }
            
            .achievement-criteria li {
                padding: 8px 0;
                border-bottom: 1px solid var(--light-gray);
                position: relative;
                padding-left: 25px;
            }
            
            .achievement-criteria li::before {
                content: '✓';
                position: absolute;
                left: 0;
                color: var(--success-color);
                font-weight: bold;
            }
            
            .achievement-criteria li:last-child {
                border-bottom: none;
            }
        </style>
    `;
    
    const bsModal = new bootstrap.Modal(modal);
    bsModal.show();
}

// Critérios das conquistas
function getAchievementCriteria(achievementKey) {
    const criteria = {
        gold: `
            <ul>
                <li>Estar entre os 3 primeiros no ranking geral</li>
                <li>Manter pontuação acima de 2500 pontos</li>
                <li>Ser ativo por pelo menos 3 meses</li>
            </ul>
        `,
        silver: `
            <ul>
                <li>Estar entre os 10 primeiros no ranking</li>
                <li>Manter pontuação acima de 1500 pontos</li>
                <li>Contribuir regularmente para a comunidade</li>
            </ul>
        `,
        bronze: `
            <ul>
                <li>Estar entre os 20 primeiros no ranking</li>
                <li>Manter pontuação acima de 800 pontos</li>
                <li>Completar pelo menos 5 atividades</li>
            </ul>
        `,
        expert: `
            <ul>
                <li>Demonstrar conhecimento avançado em sua área</li>
                <li>Receber avaliações positivas consistentes</li>
                <li>Compartilhar conteúdo de qualidade</li>
            </ul>
        `,
        mentor: `
            <ul>
                <li>Ajudar pelo menos 15 estudantes</li>
                <li>Manter alta taxa de satisfação</li>
                <li>Ser ativo na orientação por mais de 2 meses</li>
            </ul>
        `,
        contributor: `
            <ul>
                <li>Fazer mais de 30 contribuições válidas</li>
                <li>Participar ativamente em discussões</li>
                <li>Compartilhar recursos úteis</li>
            </ul>
        `
    };
    
    return criteria[achievementKey] || '<p>Critérios não disponíveis.</p>';
}

// Configurar filtros
function setupFilters() {
    const filters = ['courseFilter', 'institutionFilter', 'levelFilter', 'sortBy'];
    
    filters.forEach(filterId => {
        const filter = document.getElementById(filterId);
        if (filter) {
            filter.addEventListener('change', applyCurrentFilters);
        }
    });
}

// Aplicar filtros atuais
function applyCurrentFilters() {
    const courseFilter = document.getElementById('courseFilter')?.value || '';
    const institutionFilter = document.getElementById('institutionFilter')?.value || '';
    const levelFilter = document.getElementById('levelFilter')?.value || '';
    const sortBy = document.getElementById('sortBy')?.value || 'points';
    
    let filtered = [...(userData[currentUserType] || [])];
    
    // Aplicar filtros
    if (courseFilter) {
        filtered = filtered.filter(user => user.course === courseFilter);
    }
    
    if (institutionFilter) {
        filtered = filtered.filter(user => user.institution === institutionFilter);
    }
    
    if (levelFilter) {
        const levelRanges = {
            iniciante: [1, 3],
            intermediario: [4, 6],
            avancado: [7, 10]
        };
        
        if (levelRanges[levelFilter]) {
            const [min, max] = levelRanges[levelFilter];
            filtered = filtered.filter(user => user.level >= min && user.level <= max);
        }
    }
    
    // Aplicar ordenação
    filtered.sort((a, b) => {
        switch(sortBy) {
            case 'contributions':
                return (b.details.contributions || 0) - (a.details.contributions || 0);
            case 'level':
                return b.level - a.level;
            case 'activity':
                const activityOrder = { online: 3, recent: 2, offline: 1 };
                return activityOrder[b.activity] - activityOrder[a.activity];
            default:
                return b.points - a.points;
        }
    });
    
    filteredUsers = filtered;
    currentPage = 1;
    
    renderFilteredResults();
}

// Renderizar resultados filtrados
function renderFilteredResults() {
    const tbody = document.getElementById('rankingTableBody');
    if (!tbody) return;
    
    tbody.innerHTML = '';
    
    if (filteredUsers.length === 0) {
        tbody.innerHTML = `
            <tr>
                <td colspan="7" class="text-center py-5">
                    <div>
                        <i class="fas fa-search fa-3x text-muted mb-3"></i>
                        <h5>Nenhum usuário encontrado</h5>
                        <p class="text-muted">Tente ajustar os filtros para encontrar mais resultados.</p>
                        <button class="btn btn-orange mt-2" onclick="clearAllFilters()">
                            <i class="fas fa-refresh me-2"></i>Limpar Filtros
                        </button>
                    </div>
                </td>
            </tr>
        `;
        return;
    }
    
    const startIndex = (currentPage - 1) * itemsPerPage;
    const endIndex = startIndex + itemsPerPage;
    const pageUsers = filteredUsers.slice(startIndex, endIndex);
    
    pageUsers.forEach((user, index) => {
        const globalRank = startIndex + index + 1;
        const row = createUserRow(user, globalRank);
        tbody.appendChild(row);
    });
    
    updateTableInfo();
    updatePaginationControls();
    
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

// Limpar todos os filtros
function clearAllFilters() {
    const filters = ['courseFilter', 'institutionFilter', 'levelFilter'];
    
    filters.forEach(filterId => {
        const filter = document.getElementById(filterId);
        if (filter) filter.value = '';
    });
    
    const sortBy = document.getElementById('sortBy');
    if (sortBy) sortBy.value = 'points';
    
    applyCurrentFilters();
}

// Configurar paginação
function setupPagination() {
    const prevBtn = document.getElementById('prevPage');
    const nextBtn = document.getElementById('nextPage');
    
    if (prevBtn) {
        prevBtn.addEventListener('click', () => {
            if (currentPage > 1) {
                currentPage--;
                renderFilteredResults();
            }
        });
    }
    
    if (nextBtn) {
        nextBtn.addEventListener('click', () => {
            const totalPages = Math.ceil(filteredUsers.length / itemsPerPage);
            if (currentPage < totalPages) {
                currentPage++;
                renderFilteredResults();
            }
        });
    }
}

// Atualizar informações da tabela
function updateTableInfo() {
    const tableInfo = document.getElementById('tableInfo');
    if (!tableInfo) return;
    
    const total = filteredUsers.length;
    const start = Math.min((currentPage - 1) * itemsPerPage + 1, total);
    const end = Math.min(currentPage * itemsPerPage, total);
    
    tableInfo.textContent = `Mostrando ${start}-${end} de ${total} usuários`;
}

// Atualizar controles de paginação
function updatePaginationControls() {
    const totalPages = Math.ceil(filteredUsers.length / itemsPerPage);
    
    const prevBtn = document.getElementById('prevPage');
    const nextBtn = document.getElementById('nextPage');
    const currentPageSpan = document.getElementById('currentPage');
    const totalPagesSpan = document.getElementById('totalPages');
    
    if (prevBtn) prevBtn.disabled = currentPage <= 1;
    if (nextBtn) nextBtn.disabled = currentPage >= totalPages;
    if (currentPageSpan) currentPageSpan.textContent = currentPage;
    if (totalPagesSpan) totalPagesSpan.textContent = totalPages || 1;
}

// Configurar botões de período
function setupPeriodButtons() {
    document.querySelectorAll('.period-btn').forEach(btn => {
        btn.addEventListener('click', function() {
            document.querySelectorAll('.period-btn').forEach(b => b.classList.remove('active'));
            this.classList.add('active');
            currentPeriod = this.dataset.period;
            
            // Aqui você pode atualizar os dados baseado no período
            updatePodium();
            showAlert(`Período alterado para: ${this.textContent}`, 'info');
        });
    });
}

// Configurar controles da tabela
function setupTableControls() {
    const exportBtn = document.getElementById('exportRankingBtn');
    const refreshBtn = document.getElementById('refreshRankingBtn');
    
    if (exportBtn) {
        exportBtn.addEventListener('click', exportRankingData);
    }
    
    if (refreshBtn) {
        refreshBtn.addEventListener('click', refreshRanking);
    }
}

// Exportar dados do ranking
function exportRankingData() {
    const csvContent = generateUserCSV(filteredUsers);
    downloadCSV(csvContent, `ranking_${currentUserType}.csv`);
    showAlert('Dados exportados com sucesso!', 'success');
}

// Gerar CSV dos usuários
function generateUserCSV(data) {
    const headers = ['Posição', 'Nome', 'Nível', 'Pontos', 'Curso', 'Instituição', 'Atividade'];
    const rows = data.map((user, index) => [
        index + 1,
        user.name,
        user.level,
        user.points,
        getCourseDisplayName(user.course),
        getInstitutionDisplayName(user.institution),
        getActivityText(user.activity)
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
    const refreshBtn = document.getElementById('refreshRankingBtn');
    if (refreshBtn) {
        const icon = refreshBtn.querySelector('i');
        icon.classList.add('fa-spin');
        
        showLoading(true);
        
        setTimeout(() => {
            renderRanking();
            showLoading(false);
            icon.classList.remove('fa-spin');
            showAlert('Ranking atualizado com sucesso!', 'success');
        }, 1500);
    }
}

// Mostrar loading
function showLoading(show) {
    const overlay = document.getElementById('loadingOverlay');
    if (overlay) {
        overlay.style.display = show ? 'flex' : 'none';
        isLoading = show;
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

// Enviar mensagem
function sendMessage(userId) {
    // Buscar usuário
    let user = null;
    for (const type in userData) {
        user = userData[type].find(u => u.id === userId);
        if (user) break;
    }
    
    if (!user) {
        showAlert('Usuário não encontrado', 'error');
        return;
    }
    
    // Simular envio de mensagem
    showAlert(`Mensagem enviada para ${user.name}!`, 'success');
    
    // Em produção, abrir modal de mensagem ou redirecionar para chat
    // window.location.href = `/chat/${userId}`;
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

// Analytics e tracking
function trackUserInteraction(action, data = {}) {
    const interaction = {
        action: action,
        userType: currentUserType,
        timestamp: new Date().toISOString(),
        ...data
    };
    
    // Em produção, enviar para serviço de analytics
    console.log('User interaction tracked:', interaction);
    
    // Salvar localmente para debug
    const interactions = JSON.parse(localStorage.getItem('userInteractions') || '[]');
    interactions.push(interaction);
    
    // Manter apenas as últimas 100 interações
    if (interactions.length > 100) {
        interactions.splice(0, interactions.length - 100);
    }
    
    localStorage.setItem('userInteractions', JSON.stringify(interactions));
}

// Funcionalidades de busca
function setupQuickSearch() {
    const searchInput = document.getElementById('quickSearch');
    if (!searchInput) return;
    
    let searchTimeout;
    searchInput.addEventListener('input', function() {
        clearTimeout(searchTimeout);
        searchTimeout = setTimeout(() => {
            const query = this.value.toLowerCase().trim();
            performQuickSearch(query);
        }, 300);
    });
}

function performQuickSearch(query) {
    if (query.length === 0) {
        filteredUsers = [...(userData[currentUserType] || [])];
    } else {
        const users = userData[currentUserType] || [];
        filteredUsers = users.filter(user =>
            user.name.toLowerCase().includes(query) ||
            getCourseDisplayName(user.course).toLowerCase().includes(query) ||
            getInstitutionDisplayName(user.institution).toLowerCase().includes(query)
        );
    }
    
    currentPage = 1;
    renderFilteredResults();
    
    trackUserInteraction('search', { query: query, results: filteredUsers.length });
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
        
        .stats-list {
            display: grid;
            gap: 8px;
        }
        
        .stat-row {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 8px 0;
            border-bottom: 1px solid var(--light-gray);
        }
        
        .stat-row:last-child {
            border-bottom: none;
        }
    `;
    document.head.appendChild(style);
}

// Funcionalidades avançadas
function setupAdvancedFeatures() {
    // Atalhos de teclado
    document.addEventListener('keydown', function(e) {
        // Alt + 1,2,3 para alternar entre tipos de usuário
        if (e.altKey && ['1', '2', '3'].includes(e.key)) {
            e.preventDefault();
            const types = ['students', 'teachers', 'mentors'];
            const typeIndex = parseInt(e.key) - 1;
            if (types[typeIndex]) {
                switchUserType(types[typeIndex]);
            }
        }
        
        // Ctrl + F para busca
        if (e.ctrlKey && e.key === 'f') {
            e.preventDefault();
            const searchInput = document.getElementById('quickSearch');
            if (searchInput) searchInput.focus();
        }
        
        // Setas para navegação de páginas
        if (e.key === 'ArrowLeft' && e.ctrlKey) {
            e.preventDefault();
            const prevBtn = document.getElementById('prevPage');
            if (prevBtn && !prevBtn.disabled) prevBtn.click();
        }
        
        if (e.key === 'ArrowRight' && e.ctrlKey) {
            e.preventDefault();
            const nextBtn = document.getElementById('nextPage');
            if (nextBtn && !nextBtn.disabled) nextBtn.click();
        }
    });
    
    // Intersection Observer para lazy loading de avatares
    const imageObserver = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                const img = entry.target;
                if (img.dataset.src) {
                    img.src = img.dataset.src;
                    img.removeAttribute('data-src');
                    imageObserver.unobserve(img);
                }
            }
        });
    });
    
    // Observar imagens lazy
    document.querySelectorAll('img[data-src]').forEach(img => {
        imageObserver.observe(img);
    });
}

// Sistema de notificações
function showNotification(title, message, type = 'info') {
    // Verificar suporte a notificações
    if ('Notification' in window) {
        if (Notification.permission === 'granted') {
            new Notification(title, {
                body: message,
                icon: '/img/Academicoapp-logo3.png'
            });
        } else if (Notification.permission !== 'denied') {
            Notification.requestPermission().then(permission => {
                if (permission === 'granted') {
                    new Notification(title, {
                        body: message,
                        icon: '/img/Academicoapp-logo3.png'
                    });
                }
            });
        }
    }
    
    // Fallback para alert visual
    showAlert(message, type);
}

// Funcionalidades de debug (remover em produção)
function debugUserRanking() {
    console.log({
        currentUserType: currentUserType,
        currentPeriod: currentPeriod,
        currentPage: currentPage,
        filteredUsers: filteredUsers,
        totalUsers: Object.keys(userData).reduce((sum, type) => sum + userData[type].length, 0)
    });
}

// Simular dados adicionais para testes
function generateMockUsers(type, count = 10) {
    const mockUsers = [];
    const courses = ['informatica', 'gestao', 'direito', 'medicina', 'economia'];
    const institutions = ['uan', 'ucan', 'isc'];
    const activities = ['online', 'recent', 'offline'];
    const achievementSets = [
        ['bronze', 'contributor'],
        ['silver', 'expert'],
        ['gold', 'expert', 'mentor'],
        ['contributor'],
        ['expert', 'mentor']
    ];
    
    for (let i = 0; i < count; i++) {
        const id = Date.now() + i;
        const name = `Usuário Teste ${i + 1}`;
        const course = courses[Math.floor(Math.random() * courses.length)];
        const institution = institutions[Math.floor(Math.random() * institutions.length)];
        const level = Math.floor(Math.random() * 10) + 1;
        const points = Math.floor(Math.random() * 3000) + 500;
        const activity = activities[Math.floor(Math.random() * activities.length)];
        const achievements = achievementSets[Math.floor(Math.random() * achievementSets.length)];
        
        mockUsers.push({
            id: id,
            name: name,
            avatar: `https://via.placeholder.com/70x70/ff6b35/ffffff?text=${name.charAt(0)}`,
            level: level,
            points: points,
            course: course,
            institution: institution,
            achievements: achievements,
            activity: activity,
            details: {
                joinDate: 'Test 2024',
                completedCourses: Math.floor(Math.random() * 20),
                contributions: Math.floor(Math.random() * 100),
                helpedStudents: Math.floor(Math.random() * 50),
                averageGrade: (Math.random() * 5 + 15).toFixed(1),
                bio: `Usuário de teste gerado automaticamente para demonstração.`,
                specialties: ['Teste', 'Demo', 'Simulação']
            }
        });
    }
    
    userData[type].push(...mockUsers);
    console.log(`Adicionados ${count} usuários de teste para ${type}`);
}

// Inicialização completa
function initializeCompleteApplication() {
    addAnimationStyles();
    setupAdvancedFeatures();
    setupQuickSearch();
    
    // Track page load
    trackUserInteraction('page_load', {
        userAgent: navigator.userAgent,
        viewport: `${window.innerWidth}x${window.innerHeight}`
    });
    
    console.log('Ranking de Usuários Academicoapp inicializado com sucesso!');
    console.log('Atalhos disponíveis:');
    console.log('- Alt + 1/2/3: Alternar tipo de usuário');
    console.log('- Ctrl + F: Focar busca');
    console.log('- Ctrl + ←/→: Navegar páginas');
}

// Executar inicialização completa após DOM carregar
document.addEventListener('DOMContentLoaded', function() {
    setTimeout(initializeCompleteApplication, 200);
});

// Salvar estado antes de sair da página
window.addEventListener('beforeunload', function() {
    trackUserInteraction('page_unload', {
        timeOnPage: Date.now() - performance.timing.navigationStart,
        currentUserType: currentUserType,
        currentPage: currentPage
    });
});

// Funcionalidades experimentais (desenvolvimento)
if (typeof window !== 'undefined' && window.location.hostname === 'localhost') {
    window.debugFunctions = {
        debugRanking: debugUserRanking,
        generateMockUsers: generateMockUsers,
        switchUserType: switchUserType,
        showBestUser: showBestUser,
        trackInteraction: trackUserInteraction
    };
    
    console.log('Funções de debug disponíveis:', Object.keys(window.debugFunctions));
}// Ranking de Usuários - Academicoapp
// Dados dos usuários (simulados)
const userData = {
  students: [
    {
      id: 1,
      name: "Ana Cardoso",
      avatar: "https://via.placeholder.com/70x70/4f46e5/ffffff?text=AC",
      level: 9,
      points: 2890,
      course: "direito",
      institution: "uan",
      achievements: ["gold", "expert", "mentor", "contributor"],
      activity: "online",
      details: {
        joinDate: "Set 2022",
        completedCourses: 15,
        contributions: 67,
        helpedStudents: 34,
        averageGrade: 17.2,
        bio: "Estudante dedicada de Direito com foco em Direitos Humanos. Ativa na comunidade acadêmica.",
        specialties: ["Direitos Humanos", "Direito Civil", "Tutoria"]
      }
    },
    {
      id: 2,
      name: "Maria Santos",
      avatar: "https://via.placeholder.com/70x70/4f46e5/ffffff?text=MS",
      level: 8,
      points: 2580,
      course: "informatica",
      institution: "uan",
      achievements: ["gold", "expert", "contributor"],
      activity: "online",
      details: {
        joinDate: "Jan 2023",
        completedCourses: 12,
        contributions: 45,
        helpedStudents: 23,
        averageGrade: 16.5,
        bio: "Entusiasta de programação e inteligência artificial. Sempre disposta a ajudar colegas.",
        specialties: ["Python", "IA", "Desenvolvimento Web"]
      }
    },
    {
      id: 3,
      name: "João Pedro",
      avatar: "https://via.placeholder.com/70x70/4f46e5/ffffff?text=JP",
      level: 7,
      points: 2340,
      course: "gestao",
      institution: "ucan",
      achievements: ["silver", "mentor"],
      activity: "recent",
      details: {
        joinDate: "Mar 2023",
        completedCourses: 10,
        contributions: 38,
        helpedStudents: 19,
        averageGrade: 15.8,
        bio: "Futuro empreendedor com paixão por inovação e gestão estratégica.",
        specialties: ["Empreendedorismo", "Marketing", "Estratégia"]
      }
    }
  ],
  teachers: [
    {
      id: 4,
      name: "Prof. Carlos Mendes",
      avatar: "https://via.placeholder.com/70x70/059669/ffffff?text=CM",
      level: 10,
      points: 4520,
      course: "informatica",
      institution: "uan",
      achievements: ["gold", "expert", "mentor"],
      activity: "online",
      details: {
        joinDate: "Jan 2022",
        coursesTeaching: 5,
        studentsHelped: 156,
        publications: 23,
        rating: 4.9,
        bio: "Professor experiente em Ciência da Computação com foco em algoritmos e estruturas de dados.",
        specialties: ["Algoritmos", "Estruturas de Dados", "Programação Avançada"]
      }
    },
    {
      id: 5,
      name: "Profa. Isabel Franco",
      avatar: "https://via.placeholder.com/70x70/059669/ffffff?text=IF",
      level: 9,
      points: 4200,
      course: "gestao",
      institution: "ucan",
      achievements: ["silver", "expert"],
      activity: "recent",
      details: {
        joinDate: "Mai 2022",
        coursesTeaching: 3,
        studentsHelped: 123,
        publications: 18,
        rating: 4.7,
        bio: "Especialista em gestão empresarial com vasta experiência em consultoria.",
        specialties: ["Gestão Estratégica", "Recursos Humanos", "Consultoria"]
      }
    }
  ],
  mentors: [
    {
      id: 6,
      name: "Dr. Ricardo Silva",
      avatar: "https://via.placeholder.com/70x70/dc2626/ffffff?text=RS",
      level: 10,
      points: 3890,
      course: "medicina",
      institution: "uan",
      achievements: ["gold", "mentor", "expert"],
      activity: "online",
      details: {
        joinDate: "Ago 2021",
        mentoredStudents: 89,
        successRate: 94,
        sessionsCompleted: 234,
        specialization: "Cardiologia",
        bio: "Cardiologista experiente dedicado à formação de novos profissionais de saúde.",
        specialties: ["Cardiologia", "Medicina Interna", "Mentoria Clínica"]
      }
    }
  ]
};

// Dados de conquistas
const achievementData = {
  gold: {
    name: "Medalha de Ouro",
    description: "Usuário top 3 no ranking geral",
    icon: "fas fa-medal",
    color: "#ffd700"
  },
  silver: {
    name: "Medalha de Prata",
    description: "Usuário top 10 no ranking geral",
    icon: "fas fa-medal",
    color: "#c0c0c0"
  },
  bronze: {
    name: "Medalha de Bronze",
    description: "Usuário top 20 no ranking geral",
    icon: "fas fa-medal",
    color: "#cd7f32"
  },
  expert: {
    name: "Especialista",
    description: "Conhecimento avançado em sua área",
    icon: "fas fa-star",
    color: "#6366f1"
  },
  mentor: {
    name: "Mentor",
    description: "Ajudou mais de 15 estudantes",
    icon: "fas fa-user-graduate",
    color: "#10b981"
  },
  contributor: {
    name: "Contribuidor",
    description: "Mais de 30 contribuições ativas",
    icon: "fas fa-hands-helping",
    color: "#f59e0b"
  }
};

// Variáveis globais
let currentUserType = 'students';
let currentPeriod = 'month';
let currentPage = 1;
let itemsPerPage = 10;
let filteredUsers = [];
let isLoading = false;

// Inicialização
document.addEventListener('DOMContentLoaded', function() {
    initializeRanking();
});

function initializeRanking() {
    setupUserTypeTabs();
    updateStats();
    updatePodium();
    renderRanking();
    setupFilters();
    setupThemeToggle();
    setupPeriodButtons();
    setupPagination();
    setupTableControls();
    
    // Animar contadores após um pequeno delay
    setTimeout(animateCounters, 500);
}

// Configurar abas de tipo de usuário
function setupUserTypeTabs() {
    document.querySelectorAll('.user-type-tab').forEach(tab => {
        tab.addEventListener('click', function() {
            if (this.dataset.type) {
                switchUserType(this.dataset.type);
            }
        });
    });
}

// Alternar tipo de usuário
function switchUserType(type) {
    currentUserType = type;
    currentPage = 1;
    
    // Atualizar abas ativas
    document.querySelectorAll('.user-type-tab').forEach(tab => {
        tab.classList.toggle('active', tab.dataset.type === type);
    });
    
    // Atualizar label da tabela
    const typeLabels = {
        students: 'Estudantes',
        teachers: 'Professores',
        mentors: 'Mentores'
    };
    
    const currentTypeLabel = document.getElementById('currentTypeLabel');
    if (currentTypeLabel) {
        currentTypeLabel.textContent = typeLabels[type];
    }
    
    updateStats();
    updatePodium();
    renderRanking();
}

// Mostrar modal do melhor usuário
function showBestUser(type) {
    const users = userData[type];
    if (!users || users.length === 0) return;
    
    // Encontrar o usuário com maior pontuação
    const bestUser = users.reduce((prev, current) => 
        (prev.points > current.points) ? prev : current
    );
    
    const modal = document.getElementById('bestUserModal');
    const modalTitle = document.getElementById('bestUserModalTitle');
    const modalBody = document.getElementById('bestUserModalBody');
    const viewProfileBtn = document.getElementById('viewFullProfileBtn');
    
    if (!modal || !modalTitle || !modalBody) return;
    
    // Configurar título baseado no tipo
    const titles = {
        students: 'Melhor Estudante',
        teachers: 'Melhor Professor',
        mentors: 'Melhor Mentor'
    };
    
    modalTitle.innerHTML = `
        <i class="fas fa-crown me-2"></i>
        ${titles[type]}
    `;
    
    // Criar conteúdo do modal
    modalBody.innerHTML = createBestUserContent(bestUser, type);
    
    // Configurar botão de perfil completo
    if (viewProfileBtn) {
        viewProfileBtn.onclick = () => {
            const bsModal = bootstrap.Modal.getInstance(modal);
            if (bsModal) bsModal.hide();
            setTimeout(() => showUserProfile(bestUser.id), 300);
        };
    }
    
    // Mostrar modal
    const bsModal = new bootstrap.Modal(modal);
    bsModal.show();
    
    // Animar progress bars após o modal aparecer
    modal.addEventListener('shown.bs.modal', () => {
        animateBestUserCharts();
    }, { once: true });
}

// Criar conteúdo do modal do melhor usuário
function createBestUserContent(user, type) {
    const typeInfo = {
        students: {
            icon: 'fas fa-user-graduate',
            color: '#4f46e5',
            metrics: [
                { label: 'Cursos Completados', value: user.details.completedCourses, icon: 'fas fa-graduation-cap' },
                { label: 'Média Geral', value: user.details.averageGrade, icon: 'fas fa-star', suffix: '/20' },
                { label: 'Estudantes Ajudados', value: user.details.helpedStudents, icon: 'fas fa-hands-helping' },
                { label: 'Contribuições', value: user.details.contributions, icon: 'fas fa-plus-circle' }
            ]
        },
        teachers: {
            icon: 'fas fa-chalkboard-teacher',
            color: '#059669',
            metrics: [
                { label: 'Cursos Lecionando', value: user.details.coursesTeaching, icon: 'fas fa-book-open' },
                { label: 'Estudantes Orientados', value: user.details.studentsHelped, icon: 'fas fa-users' },
                { label: 'Publicações', value: user.details.publications, icon: 'fas fa-file-alt' },
                { label: 'Avaliação', value: user.details.rating, icon: 'fas fa-star', suffix: '/5.0' }
            ]
        },
        mentors: {
          icon: 'fas fa-user-tie',
          color: '#dc2626',
          metrics: [
            { label: 'Estudantes Mentoreados', value: user.details.mentoredStudents, icon: 'fas fa-user-friends' },
            { label: 'Taxa de Sucesso', value: user.details.successRate, icon: 'fas fa-chart-line', suffix: '%' },
            { label: 'Sessões Completadas', value: user.details.sessionsCompleted, icon: 'fas fa-calendar-check' },
            { label: 'Especialização', value: user.details.specialization, icon: 'fas fa-stethoscope', isText: true }
          ]
        }
    };
    
    const info = typeInfo[type];
    
    return `
        <div class="row">
            <div class="col-md-4 text-center mb-4">
                <div class="best-user-avatar-container">
                    <img src="${user.avatar}" alt="${user.name}" 
                         class="best-user-avatar"
                         onerror="this.src='https://via.placeholder.com/120x120/${info.color.slice(1)}/ffffff?text=${user.name.charAt(0)}'">
                    <div class="crown-badge">
                        <i class="fas fa-crown"></i>
                    </div>
                </div>
                <h4 class="mt-3 mb-2 text-primary">${user.name}</h4>
                <div class="best-user-points">
                    <span class="points-number">${user.points.toLocaleString('pt-BR')}</span>
                    <span class="points-label">pontos</span>
                </div>
                <div class="best-user-level mt-2">
                    <span class="level-badge">Nível ${user.level}</span>
                </div>
            </div>
            
            <div class="col-md-8">
                <div class="mb-4">
                    <h6 class="text-primary mb-2">
                        <i class="${info.icon} me-2"></i>Sobre
                    </h6>
                    <p class="text-muted mb-3">${user.details.bio}</p>
                    
                    <div class="specialties mb-3">
                        <small class="text-muted d-block mb-2">Especialidades:</small>
                        <div class="d-flex flex-wrap gap-2">
                            ${user.details.specialties.map(specialty => `
                                <span class="specialty-tag">${specialty}</span>
                            `).join('')}
                        </div>
                    </div>
                </div>
                
                <div class="metrics-grid">
                    ${info.metrics.map(metric => `
                        <div class="metric-item">
                            <div class="metric-icon">
                                <i class="${metric.icon}"></i>
                            </div>
                            <div class="metric-content">
                                <div class="metric-value">
                                    ${metric.isText ? metric.value : metric.value.toLocaleString('pt-BR')}${metric.suffix || ''}
                                </div>
                                <div class="metric-label">${metric.label}</div>
                            </div>
                        </div>
                    `).join('')}
                </div>
                
                <div class="achievements-section mt-4">
                    <h6 class="text-primary mb-3">
                        <i class="fas fa-trophy me-2"></i>Conquistas
                    </h6>
                    <div class="d-flex flex-wrap gap-2">
                        ${user.achievements.map(achievement => `
                            <div class="achievement-badge ${achievement}" 
                                 onclick="showAchievementDetails('${achievement}')"
                                 title="${achievementData[achievement].name}">
                                <i class="${achievementData[achievement].icon}"></i>
                            </div>
                        `).join('')}
                    </div>
                </div>
            </div>
        </div>
        
        <style>
            .best-user-avatar-container {
                position: relative;
                display: inline-block;
            }
            
            .best-user-avatar {
                width: 120px;
                height: 120px;
                border-radius: 50%;
                border: 4px solid var(--primary-orange);
                object-fit: cover;
                box-shadow: 0 8px 25px rgba(0, 0, 0, 0.2);
            }
            
            .crown-badge {
                position: absolute;
                top: -10px;
                right: 10px;
                width: 35px;
                height: 35px;
                background: linear-gradient(135deg, #ffd700, #ffed4a);
                border-radius: 50%;
                display: flex;
                align-items: center;
                justify-content: center;
                color: var(--dark-gray);
                font-size: 16px;
                box-shadow: 0 4px 15px rgba(255, 215, 0, 0.4);
                animation: bounce 2s infinite;
            }
            
            @keyframes bounce {
                0%, 100% { transform: translateY(0); }
                50% { transform: translateY(-5px); }
            }
            
            .best-user-points {
                display: flex;
                flex-direction: column;
                align-items: center;
            }
            
            .points-number {
                font-size: 2rem;
                font-weight: 700;
                color: var(--primary-orange);
                line-height: 1;
            }
            
            .points-label {
                font-size: 12px;
                color: var(--medium-gray);
                text-transform: uppercase;
                letter-spacing: 0.5px;
            }
            
            .level-badge {
                background: var(--gradient-primary);
                color: var(--white);
                padding: 6px 12px;
                border-radius: 15px;
                font-size: 12px;
                font-weight: 600;
                text-transform: uppercase;
            }
            
            .specialty-tag {
                background: rgba(255, 107, 53, 0.1);
                color: var(--primary-orange);
                padding: 4px 10px;
                border-radius: 12px;
                font-size: 12px;
                font-weight: 500;
                border: 1px solid rgba(255, 107, 53, 0.2);
            }
            
            .metrics-grid {
                display: grid;
                grid-template-columns: repeat(auto-fit, minmax(160px, 1fr));
                gap: 15px;
            }
            
            .metric-item {
                display: flex;
                align-items: center;
                gap: 12px;
                background: var(--light-gray);
                padding: 15px;
                border-radius: 12px;
                border: 1px solid rgba(255, 107, 53, 0.1);
            }
            
            .metric-icon {
                width: 40px;
                height: 40px;
                background: var(--gradient-primary);
                border-radius: 50%;
                display: flex;
                align-items: center;
                justify-content: center;
                color: var(--white);
                font-size: 16px;
                flex-shrink: 0;
            }
            
            .metric-value {
                font-size: 18px;
                font-weight: 700;
                color: var(--dark-gray);
                line-height: 1;
            }
            
            .metric-label {
                font-size: 11px;
                color: var(--medium-gray);
                text-transform: uppercase;
                letter-spacing: 0.5px;
            }
            
            .achievements-section .achievement-badge {
                width: 40px;
                height: 40px;
                border-radius: 50%;
                display: flex;
                align-items: center;
                justify-content: center;
                color: var(--white);
                font-size: 16px;
                cursor: pointer;
                transition: transform 0.2s ease;
            }
            
            .achievements-section .achievement-badge:hover {
                transform: scale(1.1);
            }
        </style>
    `;
}

// Animar gráficos do melhor usuário
function animateBestUserCharts() {
    const metricItems = document.querySelectorAll('.metric-item');
    metricItems.forEach((item, index) => {
        setTimeout(() => {
            item.style.transform = 'translateY(0)';
            item.style.opacity = '1';
        }, index * 100);
    });
}

// Atualizar estatísticas
function updateStats() {
    const statsGrid = document.getElementById('statsGrid');
    const users = userData[currentUserType] || [];
    
    let statsHtml = '';
    
    if (currentUserType === 'students') {
        statsHtml = `
            <div class="stat-card animate__animated animate__fadeInUp">
                <div class="stat-icon">
                    <i class="fas fa-user-graduate"></i>
                </div>
                <div class="stat-number" data-count="2400">0</div>
                <h5 class="stat-title">Estudantes Ativos</h5>
                <p class="stat-description">Participando ativamente na plataforma</p>
            </div>
            <div class="stat-card animate__animated animate__fadeInUp animate__delay-1s">
                <div class="stat-icon">
                    <i class="fas fa-trophy"></i>
                </div>
                <div class="stat-number" data-count="${users.reduce((sum, u) => sum + u.points, 0)}">0</div>
                <h5 class="stat-title">Pontos Totais</h5>
                <p class="stat-description">Conquistados este mês</p>
            </div>
            <div class="stat-card animate__animated animate__fadeInUp animate__delay-2s">
                <div class="stat-icon">
                    <i class="fas fa-book"></i>
                </div>
                <div class="stat-number" data-count="1200">0</div>
                <h5 class="stat-title">Cursos Completados</h5>
                <p class="stat-description">Total da comunidade</p>
            </div>
            <div class="stat-card animate__animated animate__fadeInUp animate__delay-3s">
                <div class="stat-icon">
                    <i class="fas fa-handshake"></i>
                </div>
                <div class="stat-number" data-count="850">0</div>
                <h5 class="stat-title">Colaborações</h5>
                <p class="stat-description">Estudantes ajudando estudantes</p>
            </div>
        `;
    } else if (currentUserType === 'teachers') {
        statsHtml = `
            <div class="stat-card animate__animated animate__fadeInUp">
                <div class="stat-icon">
                    <i class="fas fa-chalkboard-teacher"></i>
                </div>
                <div class="stat-number" data-count="485">0</div>
                <h5 class="stat-title">Professores Ativos</h5>
                <p class="stat-description">Educadores dedicados</p>
            </div>
            <div class="stat-card animate__animated animate__fadeInUp animate__delay-1s">
                <div class="stat-icon">
                    <i class="fas fa-users"></i>
                </div>
                <div class="stat-number" data-count="${users.reduce((sum, u) => sum + (u.details.studentsHelped || 0), 0)}">0</div>
                <h5 class="stat-title">Estudantes Orientados</h5>
                <p class="stat-description">Alunos recebendo mentoria</p>
            </div>
            <div class="stat-card animate__animated animate__fadeInUp animate__delay-2s">
                <div class="stat-icon">
                    <i class="fas fa-file-alt"></i>
                </div>
                <div class="stat-number" data-count="${users.reduce((sum, u) => sum + (u.details.publications || 0), 0)}">0</div>
                <h5 class="stat-title">Publicações</h5>
                <p class="stat-description">Artigos e pesquisas</p>
            </div>
            <div class="stat-card animate__animated animate__fadeInUp animate__delay-3s">
                <div class="stat-icon">
                    <i class="fas fa-star"></i>
                </div>
                <div class="stat-number" data-count="4.8" data-decimal="true">0</div>
                <h5 class="stat-title">Avaliação Média</h5>
                <p class="stat-description">Satisfação dos estudantes</p>
            </div>
        `;
    } else if (currentUserType === 'mentors') {
        statsHtml = `
            <div class="stat-card animate__animated animate__fadeInUp">
                <div class="stat-icon">
                    <i class="fas fa-user-tie"></i>
                </div>
                <div class="stat-number" data-count="127">0</div>
                <h5 class="stat-title">Mentores Ativos</h5>
                <p class="stat-description">Líderes da comunidade</p>
            </div>
            <div class="stat-card animate__animated animate__fadeInUp animate__delay-1s">
                <div class="stat-icon">
                    <i class="fas fa-user-friends"></i>
                </div>
                <div class="stat-number" data-count="${users.reduce((sum, u) => sum + (u.details.mentoredStudents || 0), 0)}">0</div>
                <h5 class="stat-title">Estudantes Mentoreados</h5>
                <p class="stat-description">Recebendo orientação especializada</p>
            </div>
            <div class="stat-card animate__animated animate__fadeInUp animate__delay-2s">
                <div class="stat-icon">
                    <i class="fas fa-calendar-check"></i>
                </div>
                <div class="stat-number" data-count="${users.reduce((sum, u) => sum + (u.details.sessionsCompleted || 0), 0)}">0</div>
                <h5 class="stat-title">Sessões Completadas</h5>
                <p class="stat-description">Mentorias realizadas</p>
            </div>
            <div class="stat-card animate__animated animate__fadeInUp animate__delay-3s">
                <div class="stat-icon">
                    <i class="fas fa-chart-line"></i>
                </div>
                <div class="stat-number" data-count="94">0</div>
                <h5 class="stat-title">Taxa de Sucesso (%)</h5>
                <p class="stat-description">Objetivos alcançados</p>
            </div>
        `;
    }
    
    if (statsGrid) {
        statsGrid.innerHTML = statsHtml;
        // Reativar animação de contadores
        setTimeout(animateCounters, 100);
    }
}

// Atualizar pódio
function updatePodium() {
    const podium = document.getElementById('podium');
    const users = userData[currentUserType] || [];
    
    if (!podium || users.length === 0) return;
    
    // Ordenar por pontos e pegar top 3
    const topUsers = [...users]
        .sort((a, b) => b.points - a.points)
        .slice(0, 3);
    
    let podiumHtml = '';
    
    // Pódio na ordem: 2º, 1º, 3º (visual)
    const podiumOrder = [1, 0, 2]; // índices dos usuários
    const positions = ['second', 'first', 'third'];
    const ranks = [2, 1, 3];
    
    podiumOrder.forEach((userIndex, displayIndex) => {
        const user = topUsers[userIndex];
        const position = positions[displayIndex];
        const rank = ranks[displayIndex];
        
        if (user) {
            podiumHtml += `
                <div class="podium-step ${position}" onclick="showUserProfile(${user.id})">
                    <div class="podium-rank">${rank}</div>
                    <img src="${user.avatar}" alt="${user.name}" class="podium-avatar"
                         onerror="this.src='https://via.placeholder.com/70x70/ff6b35/ffffff?text=${user.name.charAt(0)}'">
                    <div class="podium-name">${user.name}</div>
                    <div class="podium-points">${user.points.toLocaleString('pt-BR')} pts</div>
                    <div class="podium-achievements">