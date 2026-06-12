// ═══════════════════════════════════════
//  STORAGE HELPERS
// ═══════════════════════════════════════
const Storage = {
  set:    (key, value) => localStorage.setItem(key, JSON.stringify(value)),
  get:    (key)        => JSON.parse(localStorage.getItem(key)),
  remove: (key)        => localStorage.removeItem(key)
};

// ═══════════════════════════════════════
//  INDEX PAGE — PROFILE / LOGIN
// ═══════════════════════════════════════
document.addEventListener('DOMContentLoaded', () => {
  if (document.getElementById('saveProfileBtn')) {
    initLoginPage();
  }
});

function initLoginPage() {
  const profile = Storage.get('userProfile');

  if (profile && profile.name) {
    // Returning user — show welcome card
    showReturningUser(profile);
  }

  // Save profile button
  document.getElementById('saveProfileBtn').addEventListener('click', () => {
    const name       = document.getElementById('userName').value.trim();
    const age        = document.getElementById('userAge').value.trim();
    const occupation = document.getElementById('userOccupation').value.trim();
    const city       = document.getElementById('userCity').value.trim();
    const currency   = document.getElementById('userCurrency').value;
    const goalType   = document.getElementById('userGoalType').value;

    // Validation
    if (!name) {
      document.getElementById('userName').style.borderColor = 'red';
      document.getElementById('userName').placeholder      = 'Name is required!';
      return;
    }

    document.getElementById('userName').style.borderColor = '';

    // Save profile
    const profile = { name, age, occupation, city, currency, goalType };
    Storage.set('userProfile', profile);

    // Redirect to dashboard
    window.location.href = 'dashboard.html';
  });

  // Switch user button
  const switchBtn = document.getElementById('switchUserBtn');
  if (switchBtn) {
    switchBtn.addEventListener('click', () => {
      document.getElementById('returningUser').style.display = 'none';
      document.getElementById('profileForm').style.display  = 'block';

      // Clear fields for new profile
      document.getElementById('userName').value       = '';
      document.getElementById('userAge').value        = '';
      document.getElementById('userOccupation').value = '';
      document.getElementById('userCity').value       = '';
    });
  }

  // Smooth scroll for hero button
  document.querySelectorAll('a[href^="#"]').forEach(a => {
    a.addEventListener('click', e => {
      const target = document.querySelector(a.getAttribute('href'));
      if (target) {
        e.preventDefault();
        target.scrollIntoView({ behavior: 'smooth' });
      }
    });
  });
}

function showReturningUser(profile) {
  // Hide form show welcome card
  document.getElementById('profileForm').style.display   = 'none';
  document.getElementById('returningUser').style.display = 'flex';

  // Avatar
  document.getElementById('returningAvatar').textContent =
    profile.name.charAt(0).toUpperCase();

  // Name
  document.getElementById('returningName').textContent =
    `Welcome back, ${profile.name}! 👋`;

  // Meta
  const meta = [];
  if (profile.age)        meta.push(`Age ${profile.age}`);
  if (profile.occupation) meta.push(profile.occupation);
  if (profile.city)       meta.push(profile.city);
  document.getElementById('returningMeta').textContent = meta.join(' · ');

  // Goal
  const goalMap = {
    emergency:  '🛡️ Goal: Build Emergency Fund',
    savings:    '💰 Goal: Increase Savings',
    debt:       '💳 Goal: Pay Off Debt',
    investment: '📈 Goal: Start Investing',
    retirement: '🏖️ Goal: Plan for Retirement',
    home:       '🏠 Goal: Buy a Home',
    other:      '🎯 Goal: Other'
  };
  document.getElementById('returningGoal').textContent =
    goalMap[profile.goalType] || '';
}


// ═══════════════════════════════════════
//  DASHBOARD PAGE
// ═══════════════════════════════════════
document.addEventListener('DOMContentLoaded', () => {
  if (document.getElementById('welcomeTitle')) {
    initDashboard();
  }
});

function initDashboard() {
  const profile = Storage.get('userProfile');

  // If no profile redirect to login
  if (!profile || !profile.name) {
    window.location.href = 'index.html';
    return;
  }

  // Set navbar
  loadNavbar();

  // Welcome banner
  document.getElementById('welcomeTitle').textContent =
    `Welcome back, ${profile.name}! 👋`;
  document.getElementById('welcomeSub').textContent =
    `${profile.occupation || 'FinSmart User'} · ${profile.city || ''}`;

  // Date
  const now = new Date();
  document.getElementById('welcomeDate').textContent =
    now.toLocaleDateString('en-GB', { weekday:'long', day:'numeric', month:'long', year:'numeric' });

  // Score card
  const score = Storage.get('stabilityScore');
  if (score !== null) {
    document.getElementById('dashScore').textContent = `${score}/100`;
    let level = score >= 80 ? '🌟 Excellent'
              : score >= 60 ? '✅ Good'
              : score >= 40 ? '⚠️ Fair'
              : '❗ Needs Attention';
    document.getElementById('dashScoreLevel').textContent = level;
  } else {
    document.getElementById('noDataBanner').style.display = 'flex';
  }

  // Goals card
  const goals = Storage.get('goals') || [];
  if (goals.length > 0) {
    const onTrack = goals.filter(g => {
      const progress = g.target > 0
        ? Math.round((g.current / g.target) * 100) : 0;
      return progress >= 50;
    }).length;
    document.getElementById('dashGoals').textContent      = `${onTrack}/${goals.length}`;
    document.getElementById('dashGoalsSub').textContent   = 'goals on track';
  }

  // Personality card
  const type = Storage.get('personalityType');
  const personalityNames = {
    PR: '🛡️ The Protector',
    P:  '🎯 The Planner',
    O:  '🚀 The Optimizer',
    SP: '🎉 The Spender'
  };
  if (type) {
    document.getElementById('dashPersonality').textContent    = personalityNames[type] || '--';
    document.getElementById('dashPersonalitySub').textContent = 'Your financial personality';
  }

  // Savings card
  const data = Storage.get('financialData');
  if (data && data.savings) {
    const currency = profile.currency || '₹';
    document.getElementById('dashSavings').textContent    = `${currency}${Number(data.savings).toLocaleString()}`;
    document.getElementById('dashSavingsSub').textContent = 'saved per month';
  }
}


// ═══════════════════════════════════════
//  SHARED — NAVBAR LOADER
//  Call this on every inner page
// ═══════════════════════════════════════
function loadNavbar() {
  const profile = Storage.get('userProfile');
  if (!profile) return;

  const avatar = document.getElementById('navAvatar');
  const name   = document.getElementById('navName');

  if (avatar) avatar.textContent = profile.name.charAt(0).toUpperCase();
  if (name)   name.textContent   = profile.name.split(' ')[0];
}


// ═══════════════════════════════════════
//  ANALYSER PAGE
// ═══════════════════════════════════════
document.addEventListener('DOMContentLoaded', () => {
  if (document.getElementById('analyserCard')) {
    initAnalyserPage();
  }
});

function initAnalyserPage() {
  const profile = Storage.get('userProfile');
  if (!profile || !profile.name) {
    window.location.href = 'index.html';
    return;
  }

  loadNavbar();
  loadFinancialInputs();

  // Calculate button
  document.getElementById('calculateBtn').addEventListener('click', () => {
    const income        = parseFloat(document.getElementById('income').value)        || 0;
    const expenses      = parseFloat(document.getElementById('expenses').value)      || 0;
    const savings       = parseFloat(document.getElementById('savings').value)       || 0;
    const emergencyFund = parseFloat(document.getElementById('emergencyFund').value) || 0;

    if (income === 0) {
      document.getElementById('income').style.borderColor = 'red';
      document.getElementById('income').placeholder      = 'Income is required!';
      return;
    }

    document.getElementById('income').style.borderColor = '';

    // Save data
    const financialData = { income, expenses, savings, emergencyFund };
    Storage.set('financialData', financialData);

    // Calculate and save score
    const score = calculateScore(income, expenses, savings, emergencyFund);
    Storage.set('stabilityScore', score);

    // Show result
    showScoreResult(score, income, expenses, savings, emergencyFund);

    // Show next step banner
    document.getElementById('nextStepBanner').style.display = 'flex';
  });

  // Clear button
  document.getElementById('clearFinBtn').addEventListener('click', () => {
    document.getElementById('income').value        = '';
    document.getElementById('expenses').value      = '';
    document.getElementById('savings').value       = '';
    document.getElementById('emergencyFund').value = '';
    document.getElementById('scoreResult').style.display    = 'none';
    document.getElementById('savingsTip').style.display     = 'none';
    document.getElementById('nextStepBanner').style.display = 'none';
    Storage.remove('financialData');
    Storage.remove('stabilityScore');
  });
}

function loadFinancialInputs() {
  const data  = Storage.get('financialData');
  const score = Storage.get('stabilityScore');
  if (data) {
    document.getElementById('income').value        = data.income        || '';
    document.getElementById('expenses').value      = data.expenses      || '';
    document.getElementById('savings').value       = data.savings       || '';
    document.getElementById('emergencyFund').value = data.emergencyFund || '';
  }
  if (score && data) {
    showScoreResult(score, data.income, data.expenses, data.savings, data.emergencyFund);
    document.getElementById('nextStepBanner').style.display = 'flex';
  }
}

// ═══════════════════════════════════════
//  STABILITY SCORE FORMULA
// ═══════════════════════════════════════
function calculateScore(income, expenses, savings, emergencyFund) {
  let score = 0;

  const savingsRate = income > 0 ? (savings / income) * 100 : 0;
  if      (savingsRate >= 30) score += 40;
  else if (savingsRate >= 20) score += 30;
  else if (savingsRate >= 10) score += 20;
  else if (savingsRate >  0)  score += 10;

  const expenseRatio = income > 0 ? (expenses / income) * 100 : 100;
  if      (expenseRatio <= 50) score += 30;
  else if (expenseRatio <= 60) score += 22;
  else if (expenseRatio <= 70) score += 15;
  else if (expenseRatio <= 80) score += 8;

  const monthsCovered = expenses > 0 ? emergencyFund / expenses : 0;
  if      (monthsCovered >= 6) score += 30;
  else if (monthsCovered >= 4) score += 22;
  else if (monthsCovered >= 2) score += 14;
  else if (monthsCovered >= 1) score += 7;

  return Math.min(score, 100);
}

// ═══════════════════════════════════════
//  SHOW SCORE RESULT
// ═══════════════════════════════════════
function showScoreResult(score, income, expenses, savings, emergencyFund) {
  const resultBox = document.getElementById('scoreResult');
  if (!resultBox) return;
  resultBox.style.display = 'flex';

  document.getElementById('scoreNumber').textContent = score;
  drawGauge(score);

  let level, title, desc;
  if (score >= 80) {
    level = 'excellent';
    title = '🌟 Excellent Financial Health!';
    desc  = 'You are in great financial shape. Keep maintaining your savings and emergency fund.';
  } else if (score >= 60) {
    level = 'good';
    title = '✅ Good Financial Health';
    desc  = 'You are doing well! A few improvements to your savings or emergency fund will push you higher.';
  } else if (score >= 40) {
    level = 'fair';
    title = '⚠️ Fair Financial Health';
    desc  = 'There is room for improvement. Try to reduce expenses and build your emergency fund.';
  } else {
    level = 'poor';
    title = '❗ Needs Attention';
    desc  = 'Your finances need some work. Focus on cutting expenses and starting to save consistently.';
  }

  const badge = document.getElementById('healthBadge');
  badge.textContent = level.toUpperCase();
  badge.className   = `health-badge ${level}`;

  document.getElementById('healthTitle').textContent = title;
  document.getElementById('healthDesc').textContent  = desc;

  const savingsRate = income > 0 ? Math.round((savings / income) * 100) : 0;
  document.getElementById('savingsRateVal').textContent = `${savingsRate}%`;
  document.getElementById('savingsRateBar').style.width = `${Math.min(savingsRate * 2, 100)}%`;

  const expenseRatio = income > 0 ? Math.round((expenses / income) * 100) : 0;
  document.getElementById('expenseRatioVal').textContent = `${expenseRatio}%`;
  document.getElementById('expenseRatioBar').style.width = `${Math.min(expenseRatio, 100)}%`;

  const monthsCovered = expenses > 0 ? (emergencyFund / expenses).toFixed(1) : 0;
  document.getElementById('emergencyCoverageVal').textContent = `${monthsCovered} months`;
  document.getElementById('emergencyCoverageBar').style.width = `${Math.min((monthsCovered / 6) * 100, 100)}%`;

  showSavingsTip(income, expenses, savings, score);
  saveScoreHistory(score, level);

  resultBox.scrollIntoView({ behavior: 'smooth', block: 'center' });
}

// ═══════════════════════════════════════
//  DONUT GAUGE
// ═══════════════════════════════════════
function drawGauge(score) {
  const canvas = document.getElementById('scoreGauge');
  if (!canvas) return;
  const ctx = canvas.getContext('2d');
  const cx  = canvas.width  / 2;
  const cy  = canvas.height / 2;
  const r   = 65;
  const lw  = 14;

  ctx.clearRect(0, 0, canvas.width, canvas.height);

  ctx.beginPath();
  ctx.arc(cx, cy, r, 0, Math.PI * 2);
  ctx.strokeStyle = '#daeeff';
  ctx.lineWidth   = lw;
  ctx.stroke();

  const pct   = score / 100;
  const start = -Math.PI / 2;
  const end   = start + pct * Math.PI * 2;
  const grad  = ctx.createLinearGradient(cx - r, cy, cx + r, cy);
  grad.addColorStop(0, '#1a9e5c');
  grad.addColorStop(1, '#1a5fa8');

  ctx.beginPath();
  ctx.arc(cx, cy, r, start, end);
  ctx.strokeStyle = grad;
  ctx.lineWidth   = lw;
  ctx.lineCap     = 'round';
  ctx.stroke();
}

// ═══════════════════════════════════════
//  SAVINGS TIP
// ═══════════════════════════════════════
function showSavingsTip(income, expenses, savings, score) {
  const tip     = document.getElementById('savingsTip');
  const tipText = document.getElementById('savingsTipText');
  if (!tip || !tipText) return;

  const profile  = Storage.get('userProfile') || {};
  const currency = profile.currency || '₹';
  const goals    = Storage.get('goals') || [];
  let message    = '';

  if (score < 40) {
    const target = Math.round(income * 0.1);
    message = `💡 Your score is low. Try saving just ${currency}${target.toLocaleString()} per month (10% of income) to build strong habits over time.`;
  } else if (score < 60) {
    const extra = Math.round(income * 0.04);
    message = `💡 You're making progress! Saving an extra ${currency}${extra.toLocaleString()} per month could improve your score significantly within 3 months.`;
  } else if (score < 80) {
    if (goals.length > 0) {
      const g      = goals[0];
      const needed = g.target - g.current;
      const months = savings > 0 ? Math.ceil(needed / savings) : '?';
      message = `💡 At your current savings rate, you could reach your "${g.name}" goal in approximately ${months} months. Keep it up!`;
    } else {
      message = `💡 You're doing well! Consider starting an investment to grow your wealth beyond savings.`;
    }
  } else {
    message = `🌟 Excellent! You're in great financial shape. Consider diversifying into index funds or other investments to maximise long-term growth.`;
  }

  tipText.textContent = message;
  tip.style.display   = 'flex';
}

// ═══════════════════════════════════════
//  SCORE HISTORY SAVER
// ═══════════════════════════════════════
function saveScoreHistory(score, level) {
  const history = Storage.get('scoreHistory') || [];
  const now     = new Date();
  history.push({
    score, level,
    date: now.toLocaleDateString('en-GB', { day:'numeric', month:'short', year:'numeric' }),
    time: now.toLocaleTimeString('en-GB', { hour:'2-digit', minute:'2-digit' }),
    timestamp: Date.now()
  });
  if (history.length > 10) history.shift();
  Storage.set('scoreHistory', history);
}


// ═══════════════════════════════════════
//  GOALS PAGE
// ═══════════════════════════════════════
document.addEventListener('DOMContentLoaded', () => {
  if (document.getElementById('addGoalBtn')) {
    initGoalsPage();
  }
});

function initGoalsPage() {
  const profile = Storage.get('userProfile');
  if (!profile || !profile.name) {
    window.location.href = 'index.html';
    return;
  }

  loadNavbar();
  renderGoals(Storage.get('goals') || []);

  // Add goal button
  document.getElementById('addGoalBtn').addEventListener('click', () => {
    const name     = document.getElementById('goalName').value.trim();
    const target   = parseFloat(document.getElementById('goalTarget').value)  || 0;
    const current  = parseFloat(document.getElementById('goalCurrent').value) || 0;
    const date     = document.getElementById('goalDate').value;
    const category = document.getElementById('goalCategory').value;
    const notes    = document.getElementById('goalNotes').value.trim();

    // Validation
    if (!name) {
      document.getElementById('goalName').style.borderColor = 'red';
      document.getElementById('goalName').placeholder       = 'Goal name is required!';
      return;
    }
    if (target === 0) {
      document.getElementById('goalTarget').style.borderColor = 'red';
      document.getElementById('goalTarget').placeholder       = 'Target amount is required!';
      return;
    }

    document.getElementById('goalName').style.borderColor   = '';
    document.getElementById('goalTarget').style.borderColor = '';

    const goals = Storage.get('goals') || [];
    goals.push({ id: Date.now(), name, target, current, date, category, notes });
    Storage.set('goals', goals);

    // Clear form
    document.getElementById('goalName').value    = '';
    document.getElementById('goalTarget').value  = '';
    document.getElementById('goalCurrent').value = '';
    document.getElementById('goalDate').value    = '';
    document.getElementById('goalNotes').value   = '';

    renderGoals(goals);
  });
}

// ═══════════════════════════════════════
//  RENDER GOALS
// ═══════════════════════════════════════
function renderGoals(goals) {
  const list    = document.getElementById('goalsList');
  const empty   = document.getElementById('goalsEmpty');
  const summary = document.getElementById('goalsSummary');

  // Remove existing cards
  list.querySelectorAll('.goal-card').forEach(c => c.remove());

  if (goals.length === 0) {
    empty.style.display   = 'block';
    if (summary) summary.style.display = 'none';
    return;
  }

  empty.style.display = 'none';
  if (summary) summary.style.display = 'grid';

  const profile  = Storage.get('userProfile') || {};
  const currency = profile.currency || '₹';

  // Update summary
  const totalSaved  = goals.reduce((s, g) => s + g.current, 0);
  const totalTarget = goals.reduce((s, g) => s + g.target,  0);
  let   onTrack     = 0;

  if (document.getElementById('totalGoalsCount'))
    document.getElementById('totalGoalsCount').textContent = goals.length;
  if (document.getElementById('totalSavedVal'))
    document.getElementById('totalSavedVal').textContent   = `${currency}${totalSaved.toLocaleString()}`;
  if (document.getElementById('totalTargetVal'))
    document.getElementById('totalTargetVal').textContent  = `${currency}${totalTarget.toLocaleString()}`;

  goals.forEach(goal => {
    const progress = goal.target > 0
      ? Math.min(Math.round((goal.current / goal.target) * 100), 100) : 0;

    // Status
    let status, statusClass;
    if (progress >= 100) {
      status = '✅ Completed'; statusClass = 'completed';
    } else if (goal.date) {
      const daysLeft     = Math.ceil((new Date(goal.date) - new Date()) / 86400000);
      const needed       = goal.target - goal.current;
      const monthly      = Storage.get('financialData')?.savings || 0;
      const monthsNeeded = monthly > 0 ? needed / monthly : Infinity;
      const monthsAvail  = daysLeft / 30;
      if (monthsNeeded <= monthsAvail) {
        status = '✅ On Track'; statusClass = 'on-track'; onTrack++;
      } else {
        status = '⚠️ At Risk'; statusClass = 'at-risk';
      }
    } else {
      if (progress >= 50) {
        status = '✅ On Track'; statusClass = 'on-track'; onTrack++;
      } else {
        status = '⚠️ At Risk'; statusClass = 'at-risk';
      }
    }

    const card = document.createElement('div');
    card.className = 'goal-card';
    card.id        = `goal-${goal.id}`;
    card.innerHTML = `
      <div class="goal-card-top">
        <div>
          <div class="goal-card-title">${goal.name}</div>
          <div class="goal-card-meta">
            ${goal.category}${goal.notes ? ' · ' + goal.notes : ''}
          </div>
        </div>
        <span class="goal-status ${statusClass}">${status}</span>
      </div>
      <div class="goal-progress-label">
        <span>${currency}${goal.current.toLocaleString()} saved</span>
        <span>${progress}% of ${currency}${goal.target.toLocaleString()}</span>
      </div>
      <div class="goal-progress-track">
        <div class="goal-progress-fill" style="width:${progress}%"></div>
      </div>
      <div class="goal-card-bottom">
        <span class="goal-deadline">
          ${goal.date
            ? `<i class="fas fa-calendar"></i> Target: ${new Date(goal.date).toLocaleDateString('en-GB', { day:'numeric', month:'short', year:'numeric' })}`
            : '<i class="fas fa-calendar"></i> No deadline set'}
        </span>
        <div style="display:flex; gap:0.5rem;">
          <button class="goal-update-btn" onclick="toggleGoalUpdate(${goal.id})">
            <i class="fas fa-edit"></i> Update
          </button>
          <button class="goal-delete" onclick="deleteGoal(${goal.id})">
            <i class="fas fa-trash"></i> Delete
          </button>
        </div>
      </div>
      <div class="goal-update-form" id="update-form-${goal.id}" style="display:none;">
        <label>New saved amount:</label>
        <input type="number" id="update-input-${goal.id}"
               placeholder="e.g. 35000" value="${goal.current}" min="0"/>
        <button class="btn-primary"
                onclick="updateGoalAmount(${goal.id})"
                style="padding:0.5rem 1.2rem; font-size:0.85rem;">
          <i class="fas fa-save"></i> Save
        </button>
      </div>
    `;
    list.appendChild(card);
  });

  // Update on track count
  if (document.getElementById('onTrackCount'))
    document.getElementById('onTrackCount').textContent = onTrack;
}

function toggleGoalUpdate(id) {
  const form = document.getElementById(`update-form-${id}`);
  if (form) form.style.display = form.style.display === 'none' ? 'flex' : 'none';
}

function updateGoalAmount(id) {
  const input = document.getElementById(`update-input-${id}`);
  if (!input) return;
  let goals = Storage.get('goals') || [];
  goals = goals.map(g => g.id === id
    ? { ...g, current: parseFloat(input.value) || 0 } : g);
  Storage.set('goals', goals);
  renderGoals(goals);
}

function deleteGoal(id) {
  let goals = (Storage.get('goals') || []).filter(g => g.id !== id);
  Storage.set('goals', goals);
  renderGoals(goals);
}


// ═══════════════════════════════════════
//  QUIZ PAGE
// ═══════════════════════════════════════
document.addEventListener('DOMContentLoaded', () => {
  if (document.getElementById('startQuizBtn')) {
    initQuizPage();
  }
});

function initQuizPage() {
  const profile = Storage.get('userProfile');
  if (!profile || !profile.name) {
    window.location.href = 'index.html';
    return;
  }

  loadNavbar();

  // If quiz already done show complete screen
  const savedType = Storage.get('personalityType');
  if (savedType) {
    showQuizComplete(savedType);
  }

  // Start quiz button
  document.getElementById('startQuizBtn').addEventListener('click', () => {
    document.getElementById('quizStart').style.display = 'none';
    document.getElementById('quizBody').style.display  = 'block';
    currentQuestion = 0;
    answers         = [];
    showQuestion(0);
  });

  // Retake from complete screen
  const retakeBtn = document.getElementById('retakeFromCompleteBtn');
  if (retakeBtn) {
    retakeBtn.addEventListener('click', () => {
      Storage.remove('personalityType');
      Storage.remove('quizAnswers');
      document.getElementById('quizComplete').style.display = 'none';
      document.getElementById('quizStart').style.display    = 'block';
      currentQuestion = 0;
      answers         = [];
    });
  }

  // Next button
  document.getElementById('nextBtn').addEventListener('click', () => {
    if (currentQuestion < questions.length - 1) {
      currentQuestion++;
      showQuestion(currentQuestion);
    } else {
      finishQuiz();
    }
  });

  // Previous button
  document.getElementById('prevBtn').addEventListener('click', () => {
    if (currentQuestion > 0) {
      currentQuestion--;
      showQuestion(currentQuestion);
    }
  });
}

// ═══════════════════════════════════════
//  QUIZ DATA — 10 QUESTIONS
// ═══════════════════════════════════════
const questions = [
  {
    text: 'When you receive your salary, what do you do first?',
    category: 'Spending Behaviour',
    options: [
      { text: 'Save a fixed amount immediately', type: 'P'  },
      { text: 'Pay bills then spend the rest',   type: 'SP' },
      { text: 'Treat yourself first',            type: 'SP' },
      { text: 'Invest it somewhere',             type: 'O'  }
    ]
  },
  {
    text: 'How do you feel about debt or loans?',
    category: 'Debt Attitude',
    options: [
      { text: 'Avoid them completely',            type: 'PR' },
      { text: 'Only use them for emergencies',    type: 'P'  },
      { text: 'Use them for opportunities',       type: 'O'  },
      { text: 'Comfortable using regularly',      type: 'SP' }
    ]
  },
  {
    text: 'You receive unexpected extra money. What do you do?',
    category: 'Windfall Behaviour',
    options: [
      { text: 'Add it straight to savings',       type: 'PR' },
      { text: 'Spend it on something wanted',     type: 'SP' },
      { text: 'Invest it',                        type: 'O'  },
      { text: 'Pay off any existing debt first',  type: 'P'  }
    ]
  },
  {
    text: 'How often do you track your expenses?',
    category: 'Financial Discipline',
    options: [
      { text: 'Daily — I track everything',       type: 'P'  },
      { text: 'Weekly — I check in regularly',    type: 'PR' },
      { text: 'Monthly — a quick overview',       type: 'O'  },
      { text: 'Rarely or never',                  type: 'SP' }
    ]
  },
  {
    text: 'When you go shopping, you usually...',
    category: 'Shopping Habits',
    options: [
      { text: 'Stick strictly to a list',                 type: 'P'  },
      { text: 'Buy a few unplanned extras',               type: 'SP' },
      { text: 'Splurge if something catches your eye',    type: 'SP' },
      { text: 'Compare prices carefully before buying',   type: 'PR' }
    ]
  },
  {
    text: 'What is your top financial priority right now?',
    category: 'Financial Priority',
    options: [
      { text: 'Building a solid emergency fund',   type: 'PR' },
      { text: 'Buying something big I want',       type: 'SP' },
      { text: 'Growing wealth through investing',  type: 'O'  },
      { text: 'Living comfortably day to day',     type: 'SP' }
    ]
  },
  {
    text: 'How do you react when your investments drop in value?',
    category: 'Risk Attitude',
    options: [
      { text: 'Panic and consider withdrawing',    type: 'PR' },
      { text: 'Stay calm and wait it out',         type: 'P'  },
      { text: 'See it as a buying opportunity',    type: 'O'  },
      { text: 'I do not invest at all',            type: 'SP' }
    ]
  },
  {
    text: 'How much of your income do you save each month?',
    category: 'Savings Rate',
    options: [
      { text: 'More than 30%',      type: 'O'  },
      { text: 'Between 10% – 30%',  type: 'P'  },
      { text: 'Less than 10%',      type: 'PR' },
      { text: 'Nothing currently',  type: 'SP' }
    ]
  },
  {
    text: 'Do you currently have an emergency fund?',
    category: 'Emergency Preparedness',
    options: [
      { text: 'Yes — covers 6+ months of expenses',  type: 'PR' },
      { text: 'Yes — but less than 3 months',        type: 'P'  },
      { text: 'Working on building one',             type: 'O'  },
      { text: 'No — not yet',                        type: 'SP' }
    ]
  },
  {
    text: 'How do you make big financial decisions?',
    category: 'Decision Making',
    options: [
      { text: 'Research thoroughly before deciding',  type: 'P'  },
      { text: 'Go with my gut feeling',               type: 'O'  },
      { text: 'Ask friends or family first',          type: 'PR' },
      { text: 'Avoid or delay making them',           type: 'SP' }
    ]
  }
];

let currentQuestion = 0;
let answers         = [];

// ═══════════════════════════════════════
//  SHOW QUESTION
// ═══════════════════════════════════════
function showQuestion(index) {
  const q       = questions[index];
  const total   = questions.length;
  const pct     = Math.round(((index + 1) / total) * 100);

  // Update progress
  document.getElementById('quizProgressText').textContent = `Question ${index + 1} of ${total}`;
  document.getElementById('quizProgressPct').textContent  = `${pct}%`;
  document.getElementById('quizProgressFill').style.width = `${pct}%`;

  // Update question
  document.getElementById('questionNum').textContent      = `Q${index + 1}`;
  document.getElementById('questionCategory').textContent = q.category;
  document.getElementById('questionText').textContent     = q.text;

  // Prev button
  document.getElementById('prevBtn').style.display =
    index > 0 ? 'inline-flex' : 'none';

  // Next button
  const nextBtn    = document.getElementById('nextBtn');
  nextBtn.disabled = answers[index] === undefined;
  nextBtn.innerHTML = index === total - 1
    ? 'Finish <i class="fas fa-check"></i>'
    : 'Next <i class="fas fa-arrow-right"></i>';

  // Render options
  const list = document.getElementById('optionsList');
  list.innerHTML = '';

  q.options.forEach((opt, i) => {
    const btn = document.createElement('button');
    btn.className   = 'option-btn' + (answers[index] === i ? ' selected' : '');
    btn.innerHTML   = `
      <span class="option-letter">${String.fromCharCode(65 + i)}</span>
      <span class="option-text">${opt.text}</span>
    `;
    btn.addEventListener('click', () => {
      answers[index] = i;
      document.querySelectorAll('.option-btn')
        .forEach(b => b.classList.remove('selected'));
      btn.classList.add('selected');
      document.getElementById('nextBtn').disabled = false;
    });
    list.appendChild(btn);
  });
}

// ═══════════════════════════════════════
//  FINISH QUIZ
// ═══════════════════════════════════════
function finishQuiz() {
  // Count types
  const counts = { PR: 0, P: 0, O: 0, SP: 0 };
  answers.forEach((ansIndex, qIndex) => {
    const type    = questions[qIndex].options[ansIndex].type;
    counts[type]  = (counts[type] || 0) + 1;
  });

  // Dominant type
  const dominant = Object.entries(counts)
    .sort((a, b) => b[1] - a[1])[0][0];

  Storage.set('personalityType', dominant);
  Storage.set('quizAnswers', answers);

  // Show complete
  document.getElementById('quizBody').style.display     = 'none';
  showQuizComplete(dominant);
}

// ═══════════════════════════════════════
//  SHOW COMPLETE SCREEN
// ═══════════════════════════════════════
function showQuizComplete(type) {
  document.getElementById('quizStart').style.display    = 'none';
  document.getElementById('quizBody').style.display     = 'none';
  document.getElementById('quizComplete').style.display = 'block';

  const typeMap = {
    PR: { emoji: '🛡️', name: 'The Protector',  color: 'green'  },
    P:  { emoji: '🎯', name: 'The Planner',    color: 'blue'   },
    O:  { emoji: '🚀', name: 'The Optimizer',  color: 'purple' },
    SP: { emoji: '🎉', name: 'The Spender',    color: 'orange' }
  };

  const t = typeMap[type];
  const completeType = document.getElementById('quizCompleteType');
  if (completeType && t) {
    completeType.innerHTML = `
      <div class="complete-type-badge ${t.color}">
        <span>${t.emoji}</span>
        <strong>${t.name}</strong>
      </div>
    `;
  }
}


// ═══════════════════════════════════════
//  RESULTS PAGE
// ═══════════════════════════════════════
document.addEventListener('DOMContentLoaded', () => {
  if (document.getElementById('resultsEmpty')) {
    initResultsPage();
  }
});

function initResultsPage() {
  const profile = Storage.get('userProfile');
  if (!profile || !profile.name) {
    window.location.href = 'index.html';
    return;
  }

  loadNavbar();

  const type = Storage.get('personalityType');
  if (!type) {
    document.getElementById('resultsEmpty').style.display   = 'block';
    document.getElementById('resultsContent').style.display = 'none';
    return;
  }

  document.getElementById('resultsEmpty').style.display   = 'none';
  document.getElementById('resultsContent').style.display = 'block';

  buildResults(type);
}

// ═══════════════════════════════════════
//  PERSONALITY DATA
// ═══════════════════════════════════════
const personalities = {
  PR: {
    emoji: '🛡️',
    name:  'The Protector',
    tag:   'Conservative & Security-Focused',
    desc:  'You prioritise safety and security above all else. You avoid risk, build strong emergency funds and are very careful with your money. Your caution is a real strength — but it may be holding back your long-term wealth growth.',
    strengths: [
      'Maintains a strong emergency fund consistently',
      'Avoids unnecessary debt and high-risk decisions',
      'Disciplined and consistent with spending',
      'Reliable saver who rarely overspends'
    ],
    weaknesses: [
      'May miss out on wealth-building opportunities',
      'Too risk-averse for long-term financial growth',
      'Tends to keep too much cash in low-interest savings',
      'Can be slow to make important financial decisions'
    ],
    advice: [
      'Start a small low-risk investment such as an index fund or mutual fund',
      'Set a specific emergency fund target and redirect surplus after hitting it',
      'Learn about inflation risk — cash savings lose real value over time',
      'Consider a Systematic Investment Plan (SIP) for steady long-term growth',
      'Review your risk tolerance once a year as your situation changes',
      'Automate a small monthly transfer into a diversified investment account'
    ]
  },
  P: {
    emoji: '🎯',
    name:  'The Planner',
    tag:   'Organised & Goal-Driven',
    desc:  'You are methodical, disciplined and goal-oriented. You track your money carefully, set targets and stick to your plan. You are already on the right path — now it is about optimising and scaling your financial strategy further.',
    strengths: [
      'Tracks expenses consistently and accurately',
      'Sets clear financial goals and follows through',
      'Strong savings discipline and budget awareness',
      'Thinks carefully before making spending decisions'
    ],
    weaknesses: [
      'Can become overly rigid and inflexible with budgets',
      'May miss spontaneous but good financial opportunities',
      'Sometimes over-plans without taking action',
      'May not take enough calculated risks for wealth growth'
    ],
    advice: [
      'Start investing to grow your wealth beyond just saving',
      'Review and update your financial goals every 6 months',
      'Allow a small discretionary budget so planning feels less restrictive',
      'Explore tax-saving investment options available in your country',
      'Diversify your savings across different asset types',
      'Celebrate financial milestones to stay motivated long-term'
    ]
  },
  O: {
    emoji: '🚀',
    name:  'The Optimizer',
    tag:   'Strategic & Growth-Focused',
    desc:  'You think big and are always seeking growth opportunities. You are comfortable with calculated risk and keep an eye out for ways to grow your money. Your ambition is a real asset — just make sure your financial foundation is solid first.',
    strengths: [
      'Strong growth-oriented and wealth-building mindset',
      'Comfortable taking calculated and informed risks',
      'Actively seeks financial opportunities and investments',
      'High potential for long-term wealth accumulation'
    ],
    weaknesses: [
      'May neglect building a proper emergency fund',
      'Can sometimes overestimate personal risk tolerance',
      'Tendency to move quickly without enough research',
      'Risk of lifestyle inflation as income grows'
    ],
    advice: [
      'Ensure you have at least 3 months emergency fund before investing heavily',
      'Diversify your portfolio across multiple asset classes to reduce risk',
      'Set a clear maximum risk limit and commit to it in writing',
      'Document your investment strategy and review it quarterly',
      'Balance short-term investment gains with long-term financial stability',
      'Automate your emergency fund top-up before investing surplus income'
    ]
  },
  SP: {
    emoji: '🎉',
    name:  'The Spender',
    tag:   'Lifestyle-Focused & Present-Oriented',
    desc:  'You enjoy life and live in the present. You value experiences, comfort and enjoying your money today. There is nothing wrong with that — but adding a little financial structure will give you long-term security without sacrificing the fun.',
    strengths: [
      'Genuinely enjoys life and prioritises experiences',
      'Generous and giving with money towards others',
      'Not overly stressed or anxious about finances',
      'Good at finding deals, discounts and value'
    ],
    weaknesses: [
      'Little or no emergency fund built up',
      'Struggles to save consistently each month',
      'Prone to impulse spending and unplanned purchases',
      'Financially vulnerable in case of emergencies'
    ],
    advice: [
      'Start with a tiny savings goal — even 5% of income is a great beginning',
      'Use the 50/30/20 rule: 50% needs, 30% wants, 20% savings',
      'Automate your savings so the money moves before you can spend it',
      'Build a 1-month emergency fund as your very first financial goal',
      'Track all spending for just one month to identify key patterns',
      'Find free or lower-cost alternatives for things you enjoy regularly'
    ]
  }
};

// ═══════════════════════════════════════
//  BUILD RESULTS
// ═══════════════════════════════════════
function buildResults(type) {
  const p        = personalities[type];
  const score    = Storage.get('stabilityScore');
  const data     = Storage.get('financialData')  || {};
  const profile  = Storage.get('userProfile')    || {};
  const currency = profile.currency || '₹';
  const goals    = Storage.get('goals')          || [];

  // Personality card
  document.getElementById('resultEmoji').textContent = p.emoji;
  document.getElementById('resultTag').textContent   = p.tag;
  document.getElementById('resultName').textContent  = p.name;
  document.getElementById('resultDesc').textContent  = p.desc;

  // Score ring
  const ring = document.getElementById('scoreRing');
  const ringScore = document.getElementById('ringScore');
  if (score !== null && score !== undefined) {
    ringScore.textContent = score;
    const level =
      score >= 80 ? 'excellent' :
      score >= 60 ? 'good'      :
      score >= 40 ? 'fair'      : 'poor';
    ring.className = `personality-score-ring ${level}`;
  } else {
    ringScore.textContent = '--';
    ring.className = 'personality-score-ring';
  }

  // Strengths
  const sl = document.getElementById('strengthsList');
  sl.innerHTML = '';
  p.strengths.forEach(s => {
    const li = document.createElement('li');
    li.innerHTML = `<i class="fas fa-check"></i> ${s}`;
    sl.appendChild(li);
  });

  // Weaknesses
  const wl = document.getElementById('weaknessesList');
  wl.innerHTML = '';
  p.weaknesses.forEach(w => {
    const li = document.createElement('li');
    li.innerHTML = `<i class="fas fa-circle"></i> ${w}`;
    wl.appendChild(li);
  });

  // Advice grid
  const ag = document.getElementById('adviceGrid');
  ag.innerHTML = '';
  p.advice.forEach((tip, i) => {
    const item = document.createElement('div');
    item.className = 'advice-item';
    item.innerHTML = `
      <div class="advice-num">${i + 1}</div>
      <p>${tip}</p>
    `;
    ag.appendChild(item);
  });

  // Summary grid
  const sg = document.getElementById('summaryGrid');
  sg.innerHTML = '';

  const savingsRate    = data.income > 0
    ? Math.round((data.savings / data.income) * 100) : 0;
  const monthsCovered  = data.expenses > 0
    ? (data.emergencyFund / data.expenses).toFixed(1) : 0;

  const summaryItems = [
    {
      icon: 'fa-chart-pie',
      color: 'blue',
      label: 'Stability Score',
      value: score !== null ? `${score}/100` : '--'
    },
    {
      icon: 'fa-money-bill-wave',
      color: 'green',
      label: 'Monthly Income',
      value: data.income
        ? `${currency}${Number(data.income).toLocaleString()}` : '--'
    },
    {
      icon: 'fa-shopping-cart',
      color: 'red',
      label: 'Monthly Expenses',
      value: data.expenses
        ? `${currency}${Number(data.expenses).toLocaleString()}` : '--'
    },
    {
      icon: 'fa-piggy-bank',
      color: 'green',
      label: 'Monthly Savings',
      value: data.savings
        ? `${currency}${Number(data.savings).toLocaleString()}` : '--'
    },
    {
      icon: 'fa-shield-alt',
      color: 'blue',
      label: 'Emergency Fund',
      value: data.emergencyFund
        ? `${currency}${Number(data.emergencyFund).toLocaleString()}` : '--'
    },
    {
      icon: 'fa-percentage',
      color: 'purple',
      label: 'Savings Rate',
      value: data.income ? `${savingsRate}%` : '--'
    },
    {
      icon: 'fa-calendar-check',
      color: 'teal',
      label: 'Months Covered',
      value: data.expenses ? `${monthsCovered} mo` : '--'
    },
    {
      icon: 'fa-flag',
      color: 'green',
      label: 'Goals Set',
      value: goals.length > 0 ? `${goals.length} goals` : 'None yet'
    },
    {
      icon: 'fa-brain',
      color: 'purple',
      label: 'Personality',
      value: p.name
    }
  ];

  summaryItems.forEach(item => {
    const div = document.createElement('div');
    div.className = 'summary-item';
    div.innerHTML = `
      <div class="summary-icon ${item.color}">
        <i class="fas ${item.icon}"></i>
      </div>
      <div class="summary-label">${item.label}</div>
      <div class="summary-value">${item.value}</div>
    `;
    sg.appendChild(div);
  });
}


// ═══════════════════════════════════════
//  BUDGET PAGE
// ═══════════════════════════════════════
document.addEventListener('DOMContentLoaded', () => {
  if (document.getElementById('budgetEmpty')) {
    initBudgetPage();
  }
});

function initBudgetPage() {
  const profile = Storage.get('userProfile');
  if (!profile || !profile.name) {
    window.location.href = 'index.html';
    return;
  }

  loadNavbar();

  const data = Storage.get('financialData');
  if (!data || !data.income || data.income === 0) {
    document.getElementById('budgetEmpty').style.display   = 'block';
    document.getElementById('budgetContent').style.display = 'none';
    return;
  }

  document.getElementById('budgetEmpty').style.display   = 'none';
  document.getElementById('budgetContent').style.display = 'block';

  buildBudget(data, profile);
}

function buildBudget(data, profile) {
  const currency = profile.currency || '₹';
  const income   = data.income;
  const expenses = data.expenses   || 0;
  const savings  = data.savings    || 0;

  // Recommended amounts
  const recNeeds  = Math.round(income * 0.50);
  const recWants  = Math.round(income * 0.30);
  const recSaves  = Math.round(income * 0.20);

  // Display income
  document.getElementById('budgetIncomeDisplay').textContent =
    `${currency}${income.toLocaleString()}`;

  // Display amounts
  document.getElementById('budgetNeeds').textContent   =
    `${currency}${recNeeds.toLocaleString()}`;
  document.getElementById('budgetWants').textContent   =
    `${currency}${recWants.toLocaleString()}`;
  document.getElementById('budgetSavings').textContent =
    `${currency}${recSaves.toLocaleString()}`;

  // Needs actual comparison
  const needsDiff  = expenses - recNeeds;
  const needsActual = document.getElementById('needsActual');
  if (needsActual) {
    if (needsDiff > 0) {
      needsActual.innerHTML = `
        <span class="budget-over">
          <i class="fas fa-exclamation-circle"></i>
          Your actual expenses (${currency}${expenses.toLocaleString()})
          are ${currency}${needsDiff.toLocaleString()} over the recommended limit
        </span>`;
    } else if (expenses > 0) {
      needsActual.innerHTML = `
        <span class="budget-under">
          <i class="fas fa-check-circle"></i>
          Your actual expenses (${currency}${expenses.toLocaleString()})
          are within the recommended needs budget
        </span>`;
    }
  }

  // Savings actual comparison
  const savesDiff   = savings - recSaves;
  const savesActual = document.getElementById('savingsActual');
  if (savesActual) {
    if (savings === 0) {
      savesActual.innerHTML = `
        <span class="budget-over">
          <i class="fas fa-exclamation-circle"></i>
          You are not currently saving. Target: ${currency}${recSaves.toLocaleString()} per month
        </span>`;
    } else if (savesDiff >= 0) {
      savesActual.innerHTML = `
        <span class="budget-under">
          <i class="fas fa-check-circle"></i>
          Your savings (${currency}${savings.toLocaleString()})
          meet or exceed the recommended 20% target — great job!
        </span>`;
    } else {
      savesActual.innerHTML = `
        <span class="budget-warn">
          <i class="fas fa-info-circle"></i>
          Your savings (${currency}${savings.toLocaleString()}) are
          ${currency}${Math.abs(savesDiff).toLocaleString()} below
          the recommended 20% target
        </span>`;
    }
  }

  // Comparison cards
  const compGrid = document.getElementById('budgetComparisonGrid');
  if (compGrid) {
    const items = [
      {
        label:   'Recommended Needs',
        rec:     recNeeds,
        actual:  expenses,
        icon:    'fa-home',
        color:   'blue',
        currency
      },
      {
        label:   'Recommended Savings',
        rec:     recSaves,
        actual:  savings,
        icon:    'fa-piggy-bank',
        color:   'green',
        currency
      },
      {
        label:   'Available for Wants',
        rec:     recWants,
        actual:  Math.max(income - expenses - savings, 0),
        icon:    'fa-shopping-bag',
        color:   'orange',
        currency
      }
    ];

    compGrid.innerHTML = '';
    items.forEach(item => {
      const diff    = item.actual - item.rec;
      const isOver  = diff > 0;
      const pct     = item.rec > 0
        ? Math.min(Math.round((item.actual / item.rec) * 100), 200) : 0;

      const card = document.createElement('div');
      card.className = 'budget-comp-card card';
      card.innerHTML = `
        <div class="budget-comp-top">
          <div class="overview-card-icon ${item.color}">
            <i class="fas ${item.icon}"></i>
          </div>
          <div class="budget-comp-label">${item.label}</div>
        </div>
        <div class="budget-comp-values">
          <div class="budget-comp-rec">
            <small>Recommended</small>
            <strong>${item.currency}${item.rec.toLocaleString()}</strong>
          </div>
          <div class="budget-comp-arrow">→</div>
          <div class="budget-comp-actual">
            <small>Actual</small>
            <strong>${item.actual > 0
              ? item.currency + item.actual.toLocaleString()
              : '--'}</strong>
          </div>
        </div>
        <div class="budget-comp-bar">
          <div class="budget-comp-fill ${item.color}"
               style="width:${Math.min(pct, 100)}%"></div>
        </div>
        <div class="budget-comp-status ${isOver && item.actual > 0
          ? 'over' : 'ok'}">
          ${item.actual === 0
            ? '— No data yet'
            : isOver
              ? `▲ ${item.currency}${Math.abs(diff).toLocaleString()} over`
              : `✓ ${item.currency}${Math.abs(diff).toLocaleString()} under`}
        </div>
      `;
      compGrid.appendChild(card);
    });
  }

  // Tips
  buildBudgetTips(income, expenses, savings, recNeeds, recSaves, currency);
}

function buildBudgetTips(income, expenses, savings, recNeeds, recSaves, currency) {
  const tipsList = document.getElementById('budgetTipsList');
  if (!tipsList) return;

  const tips = [];
  const savingsRate = Math.round((savings / income) * 100);
  const expRatio    = Math.round((expenses / income) * 100);

  if (expRatio > 50) {
    tips.push({
      icon:  'fa-cut',
      color: 'red',
      text:  `Your expenses use ${expRatio}% of your income. Try to bring this under 50% by reviewing your largest monthly bills first.`
    });
  }

  if (savingsRate < 20) {
    const gap = recSaves - savings;
    tips.push({
      icon:  'fa-piggy-bank',
      color: 'blue',
      text:  `You are saving ${savingsRate}% of your income. Increasing savings by just ${currency}${gap.toLocaleString()} per month would hit the recommended 20% target.`
    });
  }

  if (savingsRate >= 20) {
    tips.push({
      icon:  'fa-star',
      color: 'green',
      text:  `Great work! You are hitting the recommended 20% savings target. Consider investing your surplus to grow your wealth further.`
    });
  }

  const leftover = income - expenses - savings;
  if (leftover > recNeeds * 0.4) {
    tips.push({
      icon:  'fa-lightbulb',
      color: 'orange',
      text:  `You have ${currency}${leftover.toLocaleString()} unaccounted for each month. Make sure this is intentionally allocated to wants or additional savings.`
    });
  }

  tips.push({
    icon:  'fa-calendar-check',
    color: 'blue',
    text:  'Review your budget every month and adjust whenever your income or expenses change significantly.'
  });

  tips.push({
    icon:  'fa-robot',
    color: 'green',
    text:  'Automate your savings on payday so the money moves before you have a chance to spend it.'
  });

  tipsList.innerHTML = '';
  tips.forEach(tip => {
    const item = document.createElement('div');
    item.className = 'budget-tip-item';
    item.innerHTML = `
      <div class="budget-tip-icon ${tip.color}">
        <i class="fas ${tip.icon}"></i>
      </div>
      <p>${tip.text}</p>
    `;
    tipsList.appendChild(item);
  });
}


// ═══════════════════════════════════════
//  DEBT PAGE
// ═══════════════════════════════════════
document.addEventListener('DOMContentLoaded', () => {
  if (document.getElementById('addDebtBtn')) {
    initDebtPage();
  }
});

function initDebtPage() {
  const profile = Storage.get('userProfile');
  if (!profile || !profile.name) {
    window.location.href = 'index.html';
    return;
  }

  loadNavbar();
  renderDebts(Storage.get('debts') || []);

  document.getElementById('addDebtBtn').addEventListener('click', () => {
    const name     = document.getElementById('debtName').value.trim();
    const amount   = parseFloat(document.getElementById('debtAmount').value)   || 0;
    const interest = parseFloat(document.getElementById('debtInterest').value) || 0;
    const payment  = parseFloat(document.getElementById('debtPayment').value)  || 0;
    const type     = document.getElementById('debtType').value;
    const due      = document.getElementById('debtDue').value;

    // Validation
    if (!name) {
      document.getElementById('debtName').style.borderColor = 'red';
      document.getElementById('debtName').placeholder      = 'Debt name is required!';
      return;
    }
    if (amount === 0) {
      document.getElementById('debtAmount').style.borderColor = 'red';
      document.getElementById('debtAmount').placeholder       = 'Amount is required!';
      return;
    }

    document.getElementById('debtName').style.borderColor   = '';
    document.getElementById('debtAmount').style.borderColor = '';

    const debts = Storage.get('debts') || [];
    debts.push({ id: Date.now(), name, amount, interest, payment, type, due });
    Storage.set('debts', debts);

    // Clear form
    document.getElementById('debtName').value     = '';
    document.getElementById('debtAmount').value   = '';
    document.getElementById('debtInterest').value = '';
    document.getElementById('debtPayment').value  = '';
    document.getElementById('debtDue').value      = '';

    renderDebts(debts);
  });
}

// ═══════════════════════════════════════
//  RENDER DEBTS
// ═══════════════════════════════════════
function renderDebts(debts) {
  const list    = document.getElementById('debtsList');
  const empty   = document.getElementById('debtsEmpty');
  const summary = document.getElementById('debtSummaryRow');

  // Remove existing cards
  list.querySelectorAll('.debt-card').forEach(c => c.remove());

  if (debts.length === 0) {
    if (empty)   empty.style.display   = 'block';
    if (summary) summary.style.display = 'none';
    return;
  }

  if (empty)   empty.style.display   = 'none';
  if (summary) summary.style.display = 'grid';

  const profile  = Storage.get('userProfile') || {};
  const currency = profile.currency || '₹';

  // Summary
  const totalDebt    = debts.reduce((s, d) => s + d.amount,   0);
  const totalPayment = debts.reduce((s, d) => s + d.payment,  0);
  const avgInterest  = debts.length > 0
    ? (debts.reduce((s, d) => s + d.interest, 0) / debts.length).toFixed(1)
    : 0;

  if (document.getElementById('totalDebtVal'))
    document.getElementById('totalDebtVal').textContent =
      `${currency}${totalDebt.toLocaleString()}`;
  if (document.getElementById('totalPaymentVal'))
    document.getElementById('totalPaymentVal').textContent =
      `${currency}${totalPayment.toLocaleString()}/mo`;
  if (document.getElementById('totalDebtsCount'))
    document.getElementById('totalDebtsCount').textContent = debts.length;
  if (document.getElementById('avgInterestVal'))
    document.getElementById('avgInterestVal').textContent  = `${avgInterest}%`;

  debts.forEach(debt => {
    // Payoff estimate
    let payoffText   = 'Enter monthly payment to estimate payoff';
    let payoffMonths = null;

    if (debt.payment > 0 && debt.amount > 0) {
      const mi = (debt.interest / 100) / 12;
      if (mi === 0) {
        payoffMonths = Math.ceil(debt.amount / debt.payment);
      } else {
        const val = 1 - (mi * debt.amount) / debt.payment;
        if (val > 0) {
          payoffMonths = Math.ceil(-Math.log(val) / Math.log(1 + mi));
        }
      }

      if (payoffMonths && isFinite(payoffMonths) && payoffMonths > 0) {
        const years = Math.floor(payoffMonths / 12);
        const mths  = payoffMonths % 12;
        payoffText  = `Estimated payoff: ${years > 0 ? years + ' yr ' : ''}${mths > 0 ? mths + ' mo' : ''}`;
      } else {
        payoffText = 'Monthly payment too low to pay off this debt';
      }
    }

    // Progress bar — how much is paid off (we estimate based on payments)
    const progressPct = 0; // no paid-off tracking yet — shown as 0

    // Interest type icon
    const typeIcons = {
      personal:  '💳',
      home:      '🏠',
      car:       '🚗',
      education: '🎓',
      credit:    '💰',
      other:     '📋'
    };

    const card = document.createElement('div');
    card.className = 'debt-card card';
    card.id        = `debt-${debt.id}`;
    card.innerHTML = `
      <div class="debt-card-top">
        <div class="debt-card-left">
          <div class="debt-type-icon">${typeIcons[debt.type] || '📋'}</div>
          <div>
            <div class="debt-card-title">${debt.name}</div>
            <div class="debt-card-meta">${debt.type}</div>
          </div>
        </div>
        <div class="debt-card-right">
          ${debt.interest > 0
            ? `<span class="debt-interest-badge">${debt.interest}% p.a.</span>`
            : ''}
          ${debt.due
            ? `<span class="debt-due-badge">
                <i class="fas fa-calendar"></i>
                Due: ${new Date(debt.due).toLocaleDateString('en-GB', {
                  day:'numeric', month:'short', year:'numeric'
                })}
               </span>`
            : ''}
        </div>
      </div>

      <div class="debt-stats-grid">
        <div class="debt-stat">
          <div class="debt-stat-label">Total Owed</div>
          <div class="debt-stat-value red">${currency}${debt.amount.toLocaleString()}</div>
        </div>
        <div class="debt-stat">
          <div class="debt-stat-label">Monthly Payment</div>
          <div class="debt-stat-value">
            ${debt.payment > 0
              ? currency + debt.payment.toLocaleString()
              : '-- not set'}
          </div>
        </div>
        <div class="debt-stat">
          <div class="debt-stat-label">Interest Rate</div>
          <div class="debt-stat-value">
            ${debt.interest > 0 ? debt.interest + '% p.a.' : 'No interest'}
          </div>
        </div>
        ${debt.payment > 0 && debt.interest > 0 ? `
        <div class="debt-stat">
          <div class="debt-stat-label">Monthly Interest</div>
          <div class="debt-stat-value red">
            ${currency}${Math.round((debt.amount * (debt.interest / 100)) / 12).toLocaleString()}
          </div>
        </div>` : ''}
      </div>

      <div class="debt-payoff-row">
        <div class="debt-payoff-info">
          <i class="fas fa-clock"></i>
          <span>${payoffText}</span>
        </div>
        ${payoffMonths && isFinite(payoffMonths) && payoffMonths > 0 ? `
        <div class="debt-payoff-date">
          <i class="fas fa-flag-checkered"></i>
          <span>Debt-free by: ${getPayoffDate(payoffMonths)}</span>
        </div>` : ''}
      </div>

      <div class="debt-card-actions">
        <button class="goal-delete" onclick="deleteDebt(${debt.id})">
          <i class="fas fa-trash"></i> Delete
        </button>
      </div>
    `;
    list.appendChild(card);
  });
}

// ═══════════════════════════════════════
//  HELPERS
// ═══════════════════════════════════════
function getPayoffDate(months) {
  const date = new Date();
  date.setMonth(date.getMonth() + months);
  return date.toLocaleDateString('en-GB', {
    month: 'long', year: 'numeric'
  });
}

function deleteDebt(id) {
  let debts = (Storage.get('debts') || []).filter(d => d.id !== id);
  Storage.set('debts', debts);
  renderDebts(debts);
}


// ═══════════════════════════════════════
//  HISTORY PAGE
// ═══════════════════════════════════════
document.addEventListener('DOMContentLoaded', () => {
  if (document.getElementById('historyEmpty')) {
    initHistoryPage();
  }
});

function initHistoryPage() {
  const profile = Storage.get('userProfile');
  if (!profile || !profile.name) {
    window.location.href = 'index.html';
    return;
  }

  loadNavbar();

  const history = Storage.get('scoreHistory') || [];
  renderHistory(history);

  // Clear history button
  const clearBtn = document.getElementById('clearHistoryBtn');
  if (clearBtn) {
    clearBtn.addEventListener('click', () => {
      if (confirm('Are you sure you want to clear all score history?')) {
        Storage.remove('scoreHistory');
        renderHistory([]);
      }
    });
  }
}

// ═══════════════════════════════════════
//  RENDER HISTORY
// ═══════════════════════════════════════
function renderHistory(history) {
  const empty   = document.getElementById('historyEmpty');
  const content = document.getElementById('historyContent');
  if (!empty || !content) return;

  if (history.length === 0) {
    empty.style.display   = 'block';
    content.style.display = 'none';
    return;
  }

  empty.style.display   = 'none';
  content.style.display = 'block';

  // Stats
  const scores  = history.map(h => h.score);
  const latest  = scores[scores.length - 1];
  const highest = Math.max(...scores);
  const lowest  = Math.min(...scores);

  if (document.getElementById('latestScore'))
    document.getElementById('latestScore').textContent       = `${latest}/100`;
  if (document.getElementById('highestScore'))
    document.getElementById('highestScore').textContent      = `${highest}/100`;
  if (document.getElementById('lowestScore'))
    document.getElementById('lowestScore').textContent       = `${lowest}/100`;
  if (document.getElementById('totalCalculations'))
    document.getElementById('totalCalculations').textContent = history.length;

  // Draw chart
  drawHistoryChart(history);

  // History list
  const list = document.getElementById('historyList');
  if (!list) return;
  list.innerHTML = '';

  [...history].reverse().forEach((entry, i) => {
    const isLatest = i === 0;
    const prev     = i < history.length - 1
      ? [...history].reverse()[i + 1]?.score : null;
    const diff     = prev !== null ? entry.score - prev : null;

    const item = document.createElement('div');
    item.className = 'history-item';
    item.innerHTML = `
      <div class="history-item-left">
        <div class="history-item-num">${history.length - i}</div>
        <div>
          <div class="history-item-date">
            ${entry.date}
            <span class="history-item-time">at ${entry.time}</span>
          </div>
          ${isLatest
            ? '<span class="history-latest-tag">Latest</span>'
            : ''}
        </div>
      </div>
      <div class="history-item-right">
        ${diff !== null
          ? `<span class="history-diff ${diff >= 0 ? 'up' : 'down'}">
               ${diff >= 0 ? '▲' : '▼'} ${Math.abs(diff)}
             </span>`
          : ''}
        <span class="history-item-score">${entry.score}/100</span>
        <span class="health-badge ${entry.level}">
          ${entry.level.toUpperCase()}
        </span>
      </div>
    `;
    list.appendChild(item);
  });
}

// ═══════════════════════════════════════
//  DRAW HISTORY CHART
// ═══════════════════════════════════════
function drawHistoryChart(history) {
  const canvas = document.getElementById('historyChart');
  if (!canvas) return;

  const ctx = canvas.getContext('2d');
  const W   = canvas.parentElement.offsetWidth || 800;
  const H   = 240;

  canvas.width  = W;
  canvas.height = H;
  ctx.clearRect(0, 0, W, H);

  const padL = 45;
  const padR = 20;
  const padT = 20;
  const padB = 35;

  const points = history.map((h, i) => ({
    x: padL + (i / Math.max(history.length - 1, 1)) * (W - padL - padR),
    y: padT + ((100 - h.score) / 100) * (H - padT - padB),
    score: h.score,
    date:  h.date
  }));

  // Grid lines & Y labels
  ctx.strokeStyle = '#daeeff';
  ctx.lineWidth   = 1;
  ctx.fillStyle   = '#6a96b5';
  ctx.font        = '11px DM Sans';
  ctx.textAlign   = 'right';

  [0, 25, 50, 75, 100].forEach(v => {
    const y = padT + ((100 - v) / 100) * (H - padT - padB);
    ctx.beginPath();
    ctx.moveTo(padL, y);
    ctx.lineTo(W - padR, y);
    ctx.stroke();
    ctx.fillText(v, padL - 6, y + 4);
  });

  // X labels — dates
  ctx.textAlign = 'center';
  ctx.fillStyle = '#6a96b5';
  ctx.font      = '10px DM Sans';
  points.forEach(p => {
    const shortDate = p.date.split(' ').slice(0, 2).join(' ');
    ctx.fillText(shortDate, p.x, H - 8);
  });

  if (points.length === 1) {
    // Single dot
    ctx.beginPath();
    ctx.arc(points[0].x, points[0].y, 7, 0, Math.PI * 2);
    ctx.fillStyle   = '#1a9e5c';
    ctx.strokeStyle = 'white';
    ctx.lineWidth   = 2.5;
    ctx.fill();
    ctx.stroke();

    ctx.fillStyle = '#0a2540';
    ctx.font      = 'bold 11px DM Sans';
    ctx.textAlign = 'center';
    ctx.fillText(points[0].score, points[0].x, points[0].y - 13);
    return;
  }

  // Fill area under line
  ctx.beginPath();
  ctx.moveTo(points[0].x, H - padB);
  points.forEach(p => ctx.lineTo(p.x, p.y));
  ctx.lineTo(points[points.length - 1].x, H - padB);
  ctx.closePath();
  ctx.fillStyle = 'rgba(26,158,92,0.07)';
  ctx.fill();

  // Line
  const grad = ctx.createLinearGradient(padL, 0, W - padR, 0);
  grad.addColorStop(0, '#1a9e5c');
  grad.addColorStop(1, '#1a5fa8');

  ctx.beginPath();
  ctx.moveTo(points[0].x, points[0].y);
  points.slice(1).forEach(p => ctx.lineTo(p.x, p.y));
  ctx.strokeStyle = grad;
  ctx.lineWidth   = 2.5;
  ctx.lineJoin    = 'round';
  ctx.lineCap     = 'round';
  ctx.stroke();

  // Dots and score labels
  points.forEach((p, i) => {
    const isLatest = i === points.length - 1;

    ctx.beginPath();
    ctx.arc(p.x, p.y, isLatest ? 7 : 5, 0, Math.PI * 2);
    ctx.fillStyle   = isLatest ? '#1a9e5c' : '#3b9eda';
    ctx.strokeStyle = 'white';
    ctx.lineWidth   = 2.5;
    ctx.fill();
    ctx.stroke();

    ctx.fillStyle = '#0a2540';
    ctx.font      = `${isLatest ? 'bold ' : ''}11px DM Sans`;
    ctx.textAlign = 'center';
    ctx.fillText(p.score, p.x, p.y - 13);
  });
}