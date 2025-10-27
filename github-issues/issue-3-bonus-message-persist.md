
---

### **3️⃣ github-issues/issue-3-bonus-message-persist.md**
```markdown
# Issue #3: Bonus Round Message Persistence

**Severity:** Low  
**Risk ID:** R5 (UI State)  
**Labels:** bug, ui, messaging, user-experience  

## Description
Bonus round message ("🎉 Bonus Round! Score doubled!") disappears too quickly due to timing conflicts.

## Steps to Reproduce
1. Solve 2 puzzles.  
2. Solve 3rd puzzle → bonus round triggered.  
3. Observe message timing.

## Expected Result
- Message should display for 4–5 seconds with visual effect.

## Actual Result
- Message disappears after 2–3 seconds.

## Suggested Fix
```javascript
function showMessage(msg, type='success', options={duration:5000}){
  messageEl.textContent = msg;
  messageEl.className = type;

  if(window.messageTimeout) clearTimeout(window.messageTimeout);

  window.messageTimeout = setTimeout(()=>{
    messageEl.textContent = '';
  }, options.duration);
}
