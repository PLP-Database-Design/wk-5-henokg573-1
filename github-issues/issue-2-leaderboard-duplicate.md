
---

### **2️⃣ github-issues/issue-2-leaderboard-duplicate.md**
```markdown
# Issue #2: Leaderboard Duplicate Scores

**Severity:** Low  
**Risk ID:** R1 (Data Integrity)  
**Labels:** bug, leaderboard, data-persistence  

## Description
The leaderboard can display duplicate scores when identical scores are achieved across multiple sessions.

## Steps to Reproduce
1. Play game → reach 25 points.  
2. Check leaderboard → shows [25]  
3. Play again → reach 25 points  
4. Check leaderboard → shows [25, 25] ❌  

## Expected Result
- Leaderboard should only display unique scores or include timestamps.

## Actual Result
- Duplicate entries appear without differentiation.

## Suggested Fix (With Timestamp)
```javascript
function updateLeaderboard(currentScore){
  let leaderboard = getLeaderboard();
  const timestamp = new Date().toLocaleDateString();

  leaderboard.push({
    score: currentScore,
    date: timestamp,
    id: Date.now()
  });

  leaderboard.sort((a,b)=> b.score - a.score || b.id - a.id);
  leaderboard = leaderboard.slice(0,3);
  setLeaderboard(leaderboard);
  displayLeaderboard(leaderboard);
}
