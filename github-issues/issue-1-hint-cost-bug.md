# Issue #1: Hint Cost Bug

**Severity:** Medium  
**Risk ID:** R2 (Bonus Round Logic)  
**Labels:** bug, scoring, hint-system  

## Description
Hint cost deduction does not work correctly when the player's score is less than 2 points. The game still allows hints even when the player cannot afford them.

## Steps to Reproduce
1. Start a new game (score = 0).  
2. Solve a puzzle → score = 10.  
3. Use a hint → score decreases to 8 ✅  
4. Get score to exactly 1 → use hint ❌  

## Expected Result
- Hint should not be available when score < 2.  
- Score should not go negative.

## Actual Result
- Score remains 1.  
- Hint is still usable.  
- No validation message displayed.

## Suggested Fix
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
  } else {
    showMessage("You've already used your hint for this puzzle!");
  }
}
