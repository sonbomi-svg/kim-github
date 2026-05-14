# 김비서 - 비즈니스 대시보드 프로젝트 진행 가이드

**작성일:** 2026-05-14  
**프로젝트:** Kim Secretary (김비서) - 통합 비즈니스 관리 대시보드  
**기술 스택:** HTML5, CSS3, JavaScript, Canvas API, SVG, localStorage

---

## 📋 프로젝트 개요

단일 페이지 애플리케이션(SPA)으로 구성된 비즈니스 대시보드 시스템입니다.
- **특징:** Glassmorphism 디자인, 다크/라이트 모드, CSV 데이터 통합, Canvas 차트, 보안 자격증명 저장
- **섹션:** 대시보드, 오늘의 브리핑, 회의록, 매출 현황, 업무 프로세스, 사이트 분석, 설정

---

## 🎯 진행 순서 (Step-by-Step)

### **1단계: 기초 HTML 파일 생성**

**파일:** `index.html`, `dashboard.html`

**작업 내용:**
- DOCTYPE, meta tags, 기본 구조 작성
- Glassmorphism 디자인 CSS 구현
  - `backdrop-filter: blur()`으로 배경 흐림 효과
  - `rgba(255, 255, 255, 0.7)` 등으로 반투명 카드
  - `box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1)` 그림자 효과

**주요 CSS 변수 설정:**
```css
:root {
    --bg-primary: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
    --card-bg: rgba(255, 255, 255, 0.7);
    --text-primary: #2c3e50;
    --border-color: rgba(255, 255, 255, 0.3);
}

html.dark-mode {
    --bg-primary: linear-gradient(135deg, #0f0f23 0%, #1a1a3f 100%);
    --card-bg: rgba(255, 255, 255, 0.05);
    --text-primary: #e0e0e0;
}
```

**체크리스트:**
- [ ] HTML5 기본 구조 작성
- [ ] CSS 변수 시스템 구현 (라이트/다크 모드)
- [ ] Glassmorphism 카드 스타일링

---

### **2단계: CSV 데이터 통합**

**파일:** CSV 파일들 (매출데이터.csv, 업무목록.csv, 주간일정.txt, 프로젝트현황.csv, 회의록.txt)

**작업 내용:**
- CSV 파일을 JavaScript 배열로 변환하여 HTML에 포함
- 또는 데이터를 JavaScript 객체/배열로 직접 정의

**예시 구조:**
```javascript
const salesData = [
    { date: "2026-01-05", product: "상품A", amount: 250000 },
    { date: "2026-01-05", product: "상품B", amount: 180000 },
    // ... 더 많은 데이터
];

const todoList = [
    { id: 1, title: "프로젝트 A 진행", deadline: "2026-05-17", priority: "높음", status: "진행중" },
    // ... 더 많은 항목
];
```

**데이터 집계 함수:**
```javascript
// 월별 매출 집계
function getMonthlyData() {
    const monthlyData = {};
    salesData.forEach(item => {
        const month = item.date.slice(0, 7); // "2026-01"
        monthlyData[month] = (monthlyData[month] || 0) + item.amount;
    });
    return monthlyData;
}

// 상품별 매출 집계
function getProductData() {
    const productData = {};
    salesData.forEach(item => {
        productData[item.product] = (productData[item.product] || 0) + item.amount;
    });
    return productData;
}
```

**체크리스트:**
- [ ] CSV/텍스트 데이터를 JavaScript 구조로 변환
- [ ] 데이터 집계 함수 구현 (월별, 상품별, 카테고리별 등)

---

### **3단계: 다크/라이트 모드 구현**

**위치:** `dashboard.html` 스크립트 섹션

**구현 코드:**
```javascript
function toggleTheme() {
    const html = document.documentElement;
    const isDarkMode = html.classList.toggle('dark-mode');
    localStorage.setItem('theme', isDarkMode ? 'dark' : 'light');
    updateThemeButton();
}

function updateThemeButton() {
    const button = document.querySelector('.theme-toggle');
    const isDarkMode = document.documentElement.classList.contains('dark-mode');
    button.textContent = isDarkMode ? '☀️' : '🌙';
}

// 페이지 로드 시 저장된 테마 적용
const savedTheme = localStorage.getItem('theme');
if (savedTheme === 'dark') {
    document.documentElement.classList.add('dark-mode');
}
updateThemeButton();
```

**HTML 테마 토글 버튼:**
```html
<div class="theme-toggle" onclick="toggleTheme()">🌙</div>
```

**체크리스트:**
- [ ] CSS 변수로 라이트/다크 모드 색상 정의
- [ ] localStorage로 테마 선택 저장
- [ ] 토글 버튼 구현 및 스타일링

---

### **4단계: 네비게이션 메뉴 구현 (싱글페이지 앱 구조)**

**⚠️ 중요:** 페이지 이동이 아닌 **섹션 전환** 방식 사용 (href 링크 사용 금지)

**네비게이션 HTML:**
```html
<nav class="nav-menu">
    <button class="nav-button active" onclick="switchSection('dashboard', this)">📊 대시보드</button>
    <button class="nav-button" onclick="switchSection('briefing', this)">💬 오늘의 브리핑</button>
    <button class="nav-button" onclick="switchSection('meeting', this)">📋 회의록</button>
    <button class="nav-button" onclick="switchSection('chart', this)">📈 매출 현황</button>
    <button class="nav-button" onclick="switchSection('process', this)">🔄 업무 프로세스</button>
    <button class="nav-button" onclick="switchSection('report', this)">🔍 사이트 분석</button>
    <button class="nav-button" onclick="switchSection('settings', this)">⚙️ 설정</button>
</nav>
```

**섹션 전환 함수:**
```javascript
function switchSection(sectionId, button) {
    // 모든 섹션 숨기기
    document.querySelectorAll('.content-section').forEach(section => {
        section.classList.remove('active');
    });
    // 모든 버튼 비활성화
    document.querySelectorAll('.nav-button').forEach(btn => {
        btn.classList.remove('active');
    });
    
    // 선택한 섹션 표시
    document.getElementById(sectionId + '-section').classList.add('active');
    button.classList.add('active');
    
    // 특정 섹션 로드 시 함수 실행
    if (sectionId === 'chart') {
        setTimeout(() => {
            drawLineChart();
            drawBarChart();
        }, 100);
    }
    if (sectionId === 'briefing') {
        generateBriefing();
    }
    
    window.scrollTo({ top: 0, behavior: 'smooth' });
}
```

**HTML 섹션 구조:**
```html
<div class="content-section active" id="dashboard-section">
    <!-- 대시보드 내용 -->
</div>

<div class="content-section" id="briefing-section">
    <!-- 브리핑 내용 -->
</div>

<!-- ... 다른 섹션들 ... -->
```

**CSS (섹션 표시/숨김):**
```css
.content-section {
    display: none;
}

.content-section.active {
    display: block;
    animation: fadeIn 0.3s ease-in;
}

@keyframes fadeIn {
    from { opacity: 0; transform: translateY(10px); }
    to { opacity: 1; transform: translateY(0); }
}
```

**체크리스트:**
- [ ] 네비게이션 메뉴 HTML 구현 (버튼 기반, href 아님)
- [ ] switchSection() 함수 구현
- [ ] .content-section CSS 스타일링
- [ ] 각 섹션의 기본 HTML 마크업

---

### **5단계: 보고서/분석 섹션 추가 (웹 크롤링)**

**작업 흐름:**
1. 사용자가 분석할 사이트 URL 제공
2. Chrome 브라우저에서 사이트 방문하여 내용 확인
3. 사이트의 핵심 정보를 HTML로 정리하여 대시보드에 포함

**주요 섹션:**
- 📋 **서비스 개요** — 사이트의 주요 설명
- 📂 **제품/카테고리 구성** — 제공되는 상품/서비스 분류
- 📊 **주요 특징** — 핵심 강점
- 🎯 **마케팅 포인트** — UX 측면에서의 전략

**예시 구조:**
```html
<div id="report-section" class="content-section">
    <div class="card">
        <h2>📋 서비스 개요</h2>
        <p>사이트 설명...</p>
    </div>
    
    <div class="card">
        <h2>📂 제품 카테고리</h2>
        <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(150px, 1fr)); gap: 10px;">
            <div style="background: rgba(102, 126, 234, 0.15); padding: 12px; border-radius: 8px;">
                카테고리 1
            </div>
            <!-- ... 더 많은 카테고리 ... -->
        </div>
    </div>
</div>
```

**체크리스트:**
- [ ] URL에서 사이트 정보 수집 및 분석
- [ ] 서비스 개요, 카테고리, 특징 정리
- [ ] HTML 보고서 섹션 작성

---

### **6단계: Canvas API를 이용한 차트 구현**

**선 그래프 (Line Chart) - 월별 매출:**

```javascript
function drawLineChart() {
    const canvas = document.getElementById('lineChart');
    if (!canvas) return;
    const ctx = canvas.getContext('2d');
    const monthlyData = getMonthlyData();
    const months = Object.keys(monthlyData).sort();
    const values = months.map(m => monthlyData[m]);
    
    const padding = 60;
    const chartWidth = canvas.width - 2 * padding;
    const chartHeight = canvas.height - 2 * padding;
    const maxValue = Math.max(...values);
    
    // 배경 초기화
    ctx.fillStyle = 'transparent';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    // 그리드 라인
    ctx.strokeStyle = 'rgba(0, 0, 0, 0.1)';
    ctx.lineWidth = 1;
    for (let i = 0; i <= 5; i++) {
        const y = padding + (chartHeight / 5) * i;
        ctx.beginPath();
        ctx.moveTo(padding, y);
        ctx.lineTo(canvas.width - padding, y);
        ctx.stroke();
    }
    
    // Y축 레이블
    ctx.fillStyle = '#555';
    ctx.textAlign = 'right';
    ctx.font = '12px sans-serif';
    for (let i = 0; i <= 5; i++) {
        const value = Math.round((maxValue / 5) * (5 - i));
        const y = padding + (chartHeight / 5) * i + 4;
        ctx.fillText(formatNumber(value), padding - 10, y);
    }
    
    // 축
    ctx.strokeStyle = '#ccc';
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.moveTo(padding, canvas.height - padding);
    ctx.lineTo(canvas.width - padding, canvas.height - padding);
    ctx.stroke();
    ctx.beginPath();
    ctx.moveTo(padding, padding);
    ctx.lineTo(padding, canvas.height - padding);
    ctx.stroke();
    
    // 라인 그리기
    const step = chartHeight / maxValue;
    ctx.strokeStyle = '#667eea';
    ctx.lineWidth = 3;
    ctx.beginPath();
    
    values.forEach((value, idx) => {
        const x = padding + (chartWidth / (months.length - 1 || 1)) * idx;
        const y = canvas.height - padding - value * step;
        if (idx === 0) ctx.moveTo(x, y);
        else ctx.lineTo(x, y);
    });
    ctx.stroke();
    
    // 데이터 포인트
    ctx.fillStyle = '#667eea';
    values.forEach((value, idx) => {
        const x = padding + (chartWidth / (months.length - 1 || 1)) * idx;
        const y = canvas.height - padding - value * step;
        ctx.beginPath();
        ctx.arc(x, y, 5, 0, Math.PI * 2);
        ctx.fill();
    });
    
    // X축 레이블
    ctx.fillStyle = '#555';
    ctx.textAlign = 'center';
    ctx.font = 'bold 12px sans-serif';
    months.forEach((month, idx) => {
        const x = padding + (chartWidth / (months.length - 1 || 1)) * idx;
        const y = canvas.height - padding + 20;
        ctx.fillText(month, x, y);
    });
}
```

**막대 그래프 (Bar Chart) - 상품별 매출:**

```javascript
function drawBarChart() {
    const canvas = document.getElementById('barChart');
    if (!canvas) return;
    const ctx = canvas.getContext('2d');
    const productData = getProductData();
    const products = Object.keys(productData).sort((a, b) => productData[b] - productData[a]);
    const values = products.map(p => productData[p]);
    
    // 기본 설정
    const padding = 60;
    const chartWidth = canvas.width - 2 * padding;
    const chartHeight = canvas.height - 2 * padding;
    const maxValue = Math.max(...values);
    const barWidth = chartWidth / products.length;
    const step = chartHeight / maxValue;
    
    // 배경 및 그리드
    ctx.fillStyle = 'transparent';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    ctx.strokeStyle = 'rgba(0, 0, 0, 0.1)';
    ctx.lineWidth = 1;
    for (let i = 0; i <= 5; i++) {
        const y = padding + (chartHeight / 5) * i;
        ctx.beginPath();
        ctx.moveTo(padding, y);
        ctx.lineTo(canvas.width - padding, y);
        ctx.stroke();
    }
    
    // Y축 레이블
    ctx.fillStyle = '#555';
    ctx.textAlign = 'right';
    ctx.font = '12px sans-serif';
    for (let i = 0; i <= 5; i++) {
        const value = Math.round((maxValue / 5) * (5 - i));
        const y = padding + (chartHeight / 5) * i + 4;
        ctx.fillText(formatNumber(value), padding - 10, y);
    }
    
    // 축 그리기
    ctx.strokeStyle = '#ccc';
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.moveTo(padding, canvas.height - padding);
    ctx.lineTo(canvas.width - padding, canvas.height - padding);
    ctx.stroke();
    ctx.beginPath();
    ctx.moveTo(padding, padding);
    ctx.lineTo(padding, canvas.height - padding);
    ctx.stroke();
    
    // 막대 그리기
    const colors = ['#10b981', '#059669', '#0891b2', '#0d9488', '#047857'];
    values.forEach((value, idx) => {
        const x = padding + barWidth * idx + barWidth * 0.1;
        const barHeight = value * step;
        const y = canvas.height - padding - barHeight;
        const width = barWidth * 0.8;
        
        ctx.fillStyle = colors[idx % colors.length];
        ctx.fillRect(x, y, width, barHeight);
        
        // 값 레이블
        ctx.fillStyle = '#555';
        ctx.textAlign = 'center';
        ctx.font = 'bold 11px sans-serif';
        ctx.fillText(formatNumber(value), x + width / 2, y - 5);
    });
    
    // X축 레이블
    ctx.fillStyle = '#555';
    ctx.textAlign = 'center';
    ctx.font = '11px sans-serif';
    products.forEach((product, idx) => {
        const x = padding + barWidth * idx + barWidth * 0.5;
        const y = canvas.height - padding + 35;
        const words = product.split(' ');
        words.forEach((word, i) => {
            ctx.fillText(word, x, y + i * 12);
        });
    });
}

// 숫자 포맷팅 함수
function formatNumber(num) {
    return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
}
```

**HTML Canvas 요소:**
```html
<div id="chart-section" class="content-section">
    <div class="card">
        <h2>📈 월별 매출 추이</h2>
        <canvas id="lineChart" width="800" height="400"></canvas>
        <div id="lineStats"></div>
    </div>
    
    <div class="card">
        <h2>📊 상품별 매출 비교</h2>
        <canvas id="barChart" width="800" height="400"></canvas>
        <div id="barStats"></div>
    </div>
</div>
```

**체크리스트:**
- [ ] Canvas 요소 HTML에 추가
- [ ] 월별 매출 라인 차트 구현
- [ ] 상품별 매출 막대 차트 구현
- [ ] 차트 통계 정보(총액, 평균, 최고값) 표시
- [ ] formatNumber() 함수로 숫자 포맷팅

---

### **7단계: SVG를 이용한 프로세스 다이어그램**

**5단계 프로세스 다이어그램:**

```html
<svg viewBox="0 0 800 300" style="width: 100%; height: auto; margin-top: 20px;">
    <!-- 1단계 -->
    <circle cx="50" cy="150" r="30" fill="#667eea" opacity="0.9"/>
    <text x="50" y="155" text-anchor="middle" fill="white" font-weight="bold" font-size="20">1</text>
    
    <!-- 화살표 -->
    <path d="M 80 150 L 140 150" stroke="#667eea" stroke-width="2" fill="none" marker-end="url(#arrowhead)"/>
    
    <!-- 2단계 -->
    <circle cx="170" cy="150" r="30" fill="#667eea" opacity="0.7"/>
    <text x="170" y="155" text-anchor="middle" fill="white" font-weight="bold" font-size="20">2</text>
    
    <!-- ... 계속 ... -->
    
    <!-- 라벨 -->
    <text x="50" y="200" text-anchor="middle" font-size="12">전략 수립</text>
    <text x="170" y="200" text-anchor="middle" font-size="12">데이터 수집</text>
    
    <!-- 화살표 마커 정의 -->
    <defs>
        <marker id="arrowhead" markerWidth="10" markerHeight="10" refX="9" refY="3" orient="auto">
            <polygon points="0 0, 10 3, 0 6" fill="#667eea" />
        </marker>
    </defs>
</svg>
```

**체크리스트:**
- [ ] SVG 요소로 프로세스 흐름도 구현
- [ ] 원형 노드와 화살표로 5단계 표현
- [ ] 각 단계별 라벨 추가

---

### **8단계: 오늘의 브리핑 섹션**

**기능:**
- 현재 날짜 및 요일 자동 표시
- 오늘 일정 표시
- 높은 우선순위 할 일 목록
- 진행 중인 프로젝트 진행률
- 통계 요약

**구현 코드:**

```javascript
function generateBriefing() {
    const today = new Date();
    const dateStr = today.toLocaleDateString('ko-KR', { 
        weekday: 'long', 
        year: 'numeric', 
        month: 'long', 
        day: 'numeric' 
    });
    
    // 오늘 일정 필터링
    const todaySchedule = scheduleData.filter(item => item.date === formatDate(today));
    
    // 높은 우선순위 할 일
    const highPriorityTodos = todoList.filter(item => item.priority === '높음');
    
    // 진행 중인 프로젝트
    const ongoingProjects = projectData.filter(item => item.status === '진행중');
    
    let briefingHtml = `
        <div style="padding: 20px;">
            <h2 style="color: var(--text-primary); margin-bottom: 20px;">📅 ${dateStr}</h2>
            
            <div class="card">
                <h3>📋 오늘 일정</h3>
                ${todaySchedule.length > 0 ? todaySchedule.map(item => 
                    `<div style="padding: 10px; background: rgba(102, 126, 234, 0.1); margin: 8px 0; border-radius: 6px;">
                        ${item.time} - ${item.title}
                    </div>`
                ).join('') : '<p>예정된 일정이 없습니다.</p>'}
            </div>
            
            <div class="card">
                <h3>⚡ 높은 우선순위 할 일</h3>
                ${highPriorityTodos.map(item =>
                    `<div style="padding: 10px; border-left: 4px solid #ff6b6b; background: rgba(255, 107, 107, 0.05); margin: 8px 0;">
                        <div style="font-weight: 600;">${item.title}</div>
                        <div style="font-size: 12px; color: var(--text-muted);">마감: ${item.deadline}</div>
                    </div>`
                ).join('')}
            </div>
            
            <div class="card">
                <h3>📊 진행 중인 프로젝트</h3>
                ${ongoingProjects.map(item =>
                    `<div style="margin: 15px 0;">
                        <div style="display: flex; justify-content: space-between; margin-bottom: 5px;">
                            <span style="font-weight: 600;">${item.name}</span>
                            <span style="color: var(--text-muted);">${item.progress}%</span>
                        </div>
                        <div style="background: var(--progress-bg); height: 8px; border-radius: 4px; overflow: hidden;">
                            <div style="background: linear-gradient(90deg, #667eea, #764ba2); height: 100%; width: ${item.progress}%;"></div>
                        </div>
                    </div>`
                ).join('')}
            </div>
        </div>
    `;
    
    document.getElementById('briefing-content').innerHTML = briefingHtml;
}

function formatDate(date) {
    return date.toISOString().split('T')[0];
}
```

**체크리스트:**
- [ ] 현재 날짜 자동 계산
- [ ] 오늘 일정 필터링 및 표시
- [ ] 우선순위별 할 일 분류
- [ ] 프로젝트 진행률 표시
- [ ] 통계 요약 정보 포함

---

### **9단계: 설정 섹션 - 자격증명 관리**

**구현 단계:**

1. **HTML 구조 작성:**
```html
<div id="settings-section" class="content-section">
    <div class="card">
        <h2>🔐 GitHub 토큰</h2>
        <label>토큰 입력 (localStorage에 안전하게 저장됨)</label>
        <input type="password" id="githubToken" placeholder="ghp_xxxxxxxxxxxxxxxxxxxx" 
               style="width: 100%; padding: 10px; border: 1px solid var(--border-color); 
                      background: var(--card-bg); color: var(--text-primary); border-radius: 6px;">
        <div style="display: flex; gap: 10px; margin-top: 12px;">
            <button onclick="saveGithubToken()" style="padding: 8px 16px; background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
                                                       color: white; border: none; border-radius: 6px; cursor: pointer;
                                                       font-size: 13px; font-weight: 600;">
                💾 저장
            </button>
            <button onclick="clearGithubToken()" style="padding: 8px 16px; background: rgba(255, 0, 0, 0.1);
                                                       color: #d32f2f; border: 1px solid #d32f2f; border-radius: 6px;
                                                       cursor: pointer; font-size: 13px; font-weight: 600;">
                🗑️ 삭제
            </button>
        </div>
        <div id="tokenStatus" style="margin-top: 10px; font-size: 12px; color: var(--text-secondary);"></div>
    </div>
    
    <div class="card">
        <h2>🔑 API 키</h2>
        <label>API 키 (선택사항)</label>
        <input type="password" id="apiKey" placeholder="sk_xxxxxxxxxxxxxxxx"
               style="width: 100%; padding: 10px; border: 1px solid var(--border-color);
                      background: var(--card-bg); color: var(--text-primary); border-radius: 6px;">
        <div style="display: flex; gap: 10px; margin-top: 12px;">
            <button onclick="saveApiKey()" style="padding: 8px 16px; background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
                                                  color: white; border: none; border-radius: 6px; cursor: pointer;
                                                  font-size: 13px; font-weight: 600;">
                💾 저장
            </button>
        </div>
    </div>
</div>
```

2. **JavaScript 함수 구현:**
```javascript
// GitHub 토큰 저장
function saveGithubToken() {
    const token = document.getElementById('githubToken').value.trim();
    const statusDiv = document.getElementById('tokenStatus');

    if (!token) {
        statusDiv.innerHTML = '⚠️ 토큰을 입력해주세요';
        statusDiv.style.color = '#d32f2f';
        return;
    }

    try {
        localStorage.setItem('github_token', token);
        statusDiv.innerHTML = '✅ GitHub 토큰이 저장되었습니다';
        statusDiv.style.color = '#4caf50';
        setTimeout(() => { statusDiv.innerHTML = ''; }, 3000);
    } catch (error) {
        statusDiv.innerHTML = '❌ 저장 중 오류가 발생했습니다';
        statusDiv.style.color = '#d32f2f';
    }
}

// GitHub 토큰 삭제
function clearGithubToken() {
    const statusDiv = document.getElementById('tokenStatus');
    try {
        localStorage.removeItem('github_token');
        document.getElementById('githubToken').value = '';
        statusDiv.innerHTML = '✅ GitHub 토큰이 삭제되었습니다';
        statusDiv.style.color = '#4caf50';
        setTimeout(() => { statusDiv.innerHTML = ''; }, 3000);
    } catch (error) {
        statusDiv.innerHTML = '❌ 삭제 중 오류가 발생했습니다';
        statusDiv.style.color = '#d32f2f';
    }
}

// API 키 저장
function saveApiKey() {
    const apiKey = document.getElementById('apiKey').value.trim();
    const statusDiv = document.getElementById('tokenStatus');

    if (!apiKey) {
        statusDiv.innerHTML = '⚠️ API 키를 입력해주세요';
        statusDiv.style.color = '#d32f2f';
        return;
    }

    try {
        localStorage.setItem('api_key', apiKey);
        statusDiv.innerHTML = '✅ API 키가 저장되었습니다';
        statusDiv.style.color = '#4caf50';
        setTimeout(() => { statusDiv.innerHTML = ''; }, 3000);
    } catch (error) {
        statusDiv.innerHTML = '❌ 저장 중 오류가 발생했습니다';
        statusDiv.style.color = '#d32f2f';
    }
}

// 저장된 설정 로드
function loadSettings() {
    try {
        const savedToken = localStorage.getItem('github_token');
        if (savedToken) {
            document.getElementById('githubToken').value = savedToken;
        }

        const savedApiKey = localStorage.getItem('api_key');
        if (savedApiKey) {
            document.getElementById('apiKey').value = savedApiKey;
        }
    } catch (error) {
        console.error('설정 로드 중 오류:', error);
    }
}

// 페이지 로드 시 저장된 설정 불러오기
window.addEventListener('DOMContentLoaded', function() {
    loadSettings();
});
```

**체크리스트:**
- [ ] GitHub 토큰 입력 필드 구현
- [ ] API 키 입력 필드 구현 (선택사항)
- [ ] localStorage에 저장/로드 함수 구현
- [ ] 사용자 피드백 메시지 표시
- [ ] 페이지 로드 시 자동으로 저장된 값 복구

---

### **10단계: .env.local 파일 생성**

**파일명:** `.env.local` (프로젝트 루트)

**내용:**
```env
# GitHub Token
# 깃허브 토큰을 아래에 입력해주세요
GITHUB_TOKEN=ghp_YourTokenHere

# API 설정
# 필요한 API 키들을 여기에 추가할 수 있습니다
API_KEY=

# 애플리케이션 설정
APP_NAME=김비서
APP_VERSION=1.0.0
```

**목적:**
- 로컬 환경에서 민감한 정보 관리
- GitHub 푸시 시 .gitignore로 보호
- 개발자가 직접 토큰 입력 가능

**체크리스트:**
- [ ] .env.local 파일 생성
- [ ] 템플릿 작성 (빈 토큰 필드 포함)
- [ ] 파일은 .gitignore에 포함되도록 확인

---

### **11단계: Git 저장소 초기화 및 GitHub 푸시**

**Step 1: Git 초기화**
```bash
cd "프로젝트폴더"
git init
```

**Step 2: .gitignore 파일 생성**
```bash
# .gitignore 파일 내용
.env
.env.local
.env.*.local
.vscode/
.idea/
*.log
node_modules/
.claude/
```

**Step 3: Git 사용자 설정**
```bash
git config user.email "your-email@gmail.com"
git config user.name "your-github-username"
```

**Step 4: 리모트 저장소 연결**
```bash
git remote add origin https://github.com/your-username/your-repo.git
```

**Step 5: 모든 파일 추가 및 커밋**
```bash
git add .
git commit -m "Initial commit: [프로젝트 설명]"
```

**Step 6: GitHub에 푸시**
```bash
# 방법 1: Personal Access Token 사용
GITHUB_TOKEN=$(grep GITHUB_TOKEN .env.local | cut -d'=' -f2)
git push -u https://your-username:${GITHUB_TOKEN}@github.com/your-username/your-repo.git master

# 방법 2: SSH 키 설정된 경우
git push -u origin master
```

**체크리스트:**
- [ ] git init으로 저장소 초기화
- [ ] .gitignore 파일 작성 (.env.local 포함)
- [ ] git config로 사용자 정보 설정
- [ ] 리모트 저장소 연결
- [ ] 모든 파일 commit
- [ ] GitHub에 성공적으로 푸시 완료

---

## 📁 최종 프로젝트 구조

```
프로젝트폴더/
├── dashboard.html              # 메인 대시보드 (싱글페이지 앱)
├── index.html                  # 소개 페이지
├── chart.html                  # 차트 참조 파일 (대시보드에 통합)
├── report.html                 # 분석 리포트 참조
├── report_hyponicmall.html     # 사이트 분석 예시
├── meeting-result.html         # 회의록 참조
├── diagram.svg                 # 프로세스 다이어그램
├── .env.local                  # 환경 변수 (로컬만, Git 제외)
├── .gitignore                  # Git 무시 파일 목록
├── 매출데이터.csv              # 판매 데이터
├── 업무목록.csv                # 할 일 목록
├── 주간일정.txt                # 주간 일정
├── 프로젝트현황.csv            # 진행 중인 프로젝트
├── 회의록.txt                  # 회의 기록
└── .git/                       # Git 저장소 (자동 생성)
```

---

## 🔑 핵심 개념 정리

### **Glassmorphism 디자인**
- 반투명 배경 (rgba) + 배경 흐림 (backdrop-filter: blur)
- 계층감 있는 그림자 효과
- 색상: 파란색 계열 (#667eea, #764ba2) + 반투명 흰색

### **싱글페이지 앱 (SPA) 구조**
- 페이지 이동 대신 섹션 전환
- 버튼 onclick으로 switchSection() 함수 호출
- 모든 내용이 하나의 HTML 파일에 포함

### **데이터 집계**
- CSV 데이터를 JavaScript 배열로 변환
- reduce(), map(), filter() 함수로 데이터 분석
- 월별, 상품별, 카테고리별 그룹핑

### **로컬스토리지 보안**
- password 입력 필드로 마스킹
- localStorage는 브라우저 로컬에만 저장
- .env.local은 .gitignore로 보호

### **Canvas API 차트**
- 좌표 계산: padding과 스케일 조정
- 그리드, 축, 라벨 수동 그리기
- 외부 라이브러리 없이 순수 Canvas 구현

---

## 🎓 다음 프로젝트에서 적용하기

다음 프로젝트에서는 이 가이드를 참고하여:

1. **Step 1-4:** 기초 HTML + 다크모드 + 네비게이션 구현 (필수)
2. **Step 5-8:** 데이터 통합, 차트, 다이어그램 추가 (필요시)
3. **Step 9-11:** 설정 섹션, Git 설정, 푸시 (마무리)

**빠른 시작 체크리스트:**
```
[ ] 1. HTML 기본 구조 + Glassmorphism 스타일
[ ] 2. CSS 변수로 라이트/다크 모드
[ ] 3. localStorage로 테마 저장
[ ] 4. 네비게이션 + switchSection() 함수
[ ] 5. 데이터 배열 정의
[ ] 6. Canvas 차트 (필요시)
[ ] 7. SVG 다이어그램 (필요시)
[ ] 8. 설정 섹션 + localStorage 저장
[ ] 9. .env.local + .gitignore
[ ] 10. Git init + commit + push
```

---

**작성자:** Claude Code  
**최종 업데이트:** 2026-05-14  
**버전:** 1.0
