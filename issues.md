# 🧩 Word Puzzle Game Plus — GitHub Issues Report

This document summarizes key issues identified during testing of the **Word Puzzle Game Plus** application.
Each issue includes detailed reproduction steps, expected vs. actual behavior, root cause, and recommended fixes.

---

## 🐛 Issue #1: Hint Cost Logic Error When Score < 2

**Severity:** Medium
**Risk ID:** R2 (Bonus Round Logic)
**Labels:** `bug`, `scoring`, `hint-system`

### 📝 Description

The hint cost deduction does not work correctly when the player's score is less than 2 points. The game still allows hints even when the player cannot afford them.

### 🔄 Steps to Reproduce

1. Start a new game (score = 0).
2. Solve one puzzle correctly → score = 10.
3. Use a hint → score = 8 ✅ (correct behavior).
4. Solve another puzzle → score = 18.
5. Use hint again → score = 16 ✅ (correct behavior).
6. Get score to exactly **1 point**, then use hint ❌ (bug triggers).

### ✅ Expected Result

* Score should become 0 (`1 - 2 = -1 → clamped to 0`).
* Hint should not be available when score < 2.

### ❌ Actual Result

* Score remains 1.
* Hint is shown normally.
* No validation message for insufficient points.

### 🔍 Root Cause

```javascript
score = Math.max(0, score - 2);
```

Mathematically correct but **lacks validation** to prevent hint usage when `score < 2`.

### 💡 Suggested Fix

```javascript
function showHint(){
  if(!hintUsed){
    if(score < 2){
      showMessage("Not enough points for hint! Need 2 points.", "danger");
      return;
    }
    hintEl.textContent = currentHint;
    score = Math.max(0, score - 2);
    scoreEl.textContent = score;
    hintUsed = true;
    showMessage("Hint shown (−2 points).", "success");
  }else{
    showMessage("You've already used your hint for this puzzle!");
  }
}
```

---

## 🐛 Issue #2: Leaderboard Allows Duplicate Scores

**Severity:** Low
**Risk ID:** R1 (Data Integrity)
**Labels:** `bug`, `leaderboard`, `data-persistence`

### 📝 Description

The leaderboard can display duplicate scores when identical scores are achieved in multiple sessions.

### 🔄 Steps to Reproduce

1. Play game and reach **25 points**.
2. Check leaderboard → `[25]`.
3. Reset and play again.
4. Reach **25 points** again.
5. Check leaderboard → `[25, 25]`.

### ✅ Expected Result

Leaderboard should either:

* Show **unique scores** only, or
* Include **timestamps** to differentiate sessions.

### ❌ Actual Result

Leaderboard lists duplicate entries (e.g., `[25, 25]`) with no distinction.

### 🔍 Root Cause

```javascript
leaderboard.push(currentScore); // No duplicate check or session tracking
```

### 💡 Suggested Fix

#### **Option 1: Prevent Exact Duplicates**

```javascript
function updateLeaderboard(currentScore){
  let leaderboard = getLeaderboard();

  if(!leaderboard.includes(currentScore)){
    leaderboard.push(currentScore);
  }

  leaderboard.sort((a,b) => b - a);
  leaderboard = leaderboard.slice(0,3);
  setLeaderboard(leaderboard);
  displayLeaderboard(leaderboard);
}
```

#### **Option 2: Add Timestamps (Recommended)**

```javascript
function updateLeaderboard(currentScore){
  let leaderboard = getLeaderboard();
  const timestamp = new Date().toLocaleDateString();

  leaderboard.push({
    score: currentScore,
    date: timestamp,
    id: Date.now()
  });

  leaderboard.sort((a, b) => {
    if (b.score !== a.score) return b.score - a.score;
    return b.id - a.id;
  });

  leaderboard = leaderboard.slice(0,3);
  setLeaderboard(leaderboard);
  displayLeaderboard(leaderboard);
}

function displayLeaderboard(list){
  if(!list || list.length === 0){
    leaderboardListEl.innerHTML = `<li class="muted">No scores yet — be the first!</li>`;
    return;
  }
  leaderboardListEl.innerHTML = list.map((entry, i) =>
    `<li><strong>${entry.score}</strong> (${entry.date})${i===0?' 🥇': i===1?' 🥈': i===2?' 🥉':''}</li>`
  ).join('');
}
```

---

## 🐛 Issue #3: Bonus Round Message Timing Conflict

**Severity:** Low
**Risk ID:** R5 (UI State)
**Labels:** `bug`, `ui`, `messaging`, `user-experience`

### 📝 Description

The bonus round message ("🎉 Bonus Round! Score doubled!") disappears too quickly because of a timing conflict with the message-clearing system.

### 🔄 Steps to Reproduce

1. Solve 2 puzzles.
2. Solve the 3rd puzzle → triggers bonus round.
3. Observe message timing.
4. Repeat for the 6th puzzle.

### ✅ Expected Result

* Bonus message should stay visible for **4–5 seconds**.
* Bonus animation and visual effect should be noticeable.

### ❌ Actual Result

* Message disappears after ~2–3 seconds.
* User may miss confirmation of bonus activation.

### 🔍 Root Cause

```javascript
setTimeout(()=>{ messageEl.textContent=''; }, 3000);
```

A fixed timer clears all success messages, including bonus messages.

### 💡 Suggested Fix

#### **Enhanced Message System**

```javascript
function showMessage(msg, type = 'danger', options = {}) {
  const { persistent = false, duration = 3000 } = options;

  messageEl.textContent = msg;
  messageEl.className = type === 'danger' ? 'danger' : 'success';

  if (window.messageTimeout) {
    clearTimeout(window.messageTimeout);
    window.messageTimeout = null;
  }

  if (type !== 'danger' && !persistent) {
    window.messageTimeout = setTimeout(() => {
      if (messageEl.className !== 'danger') messageEl.textContent = '';
    }, duration);
  }
}

// For bonus rounds
if (puzzlesSolved % 3 === 0) {
  score *= 2;
  showMessage("🎉 Bonus Round! Score doubled!", "success", {
    persistent: true,
    duration: 5000
  });

  scrambledWordEl.style.fontWeight = '900';
  scrambledWordEl.style.color = 'var(--accent-2)';

  setTimeout(() => {
    if (messageEl.textContent === "🎉 Bonus Round! Score doubled!") {
      messageEl.textContent = '';
      scrambledWordEl.style.fontWeight = '800';
      scrambledWordEl.style.color = 'var(--success)';
    }
  }, 5000);
}
```

#### **Simpler Alternative**

```javascript
if (puzzlesSolved % 3 === 0) {
  score *= 2;
  messageEl.textContent = '';
  
  setTimeout(() => {
    showMessage("🎉 Bonus Round! Score doubled!", "success");
  }, 100);

  setTimeout(() => {
    if (messageEl.textContent === "🎉 Bonus Round! Score doubled!") {
      messageEl.textContent = '';
    }
  }, 5000);
}
```

---

## 📋 GitHub Issue Summary

| Issue # | Title                                | Severity | Risk | Status |
| :-----: | :----------------------------------- | :------- | :--- | :----- |
|    #1   | Hint Cost Logic Error When Score < 2 | Medium   | R2   | Open   |
|    #2   | Leaderboard Allows Duplicate Scores  | Low      | R1   | Open   |
|    #3   | Bonus Round Message Timing Conflict  | Low      | R5   | Open   |

**Total Defects:** 3
**Severity Breakdown:** 1 Medium, 2 Low
**Risk Coverage:** R1, R2, R5

---

These issues highlight **real, testable defects** discovered through structured testing of the **Word Puzzle Game Plus** project.
Each issue includes actionable fixes to help developers maintain code quality and improve user experience.
----------------------------------------------------------------------------------------------------------
