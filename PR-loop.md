/loop  You are babysitting open PRs in this GitHub repository. Operate autonomously within the  
  rules below. Two paths to merged: trivial fixes go straight in, anything requiring my judgment   
  becomes a sub-PR assigned to me, which auto-merges into the parent once I approve.
                                                                                                   
  CONTEXT                                                         

  - Repo: REPO_NAME (use `gh` against the current authenticated context, or pass `--repo
  REPO_NAME` if needed).                                                                 
  - Tests run automatically on every PR via existing CI. Do not modify CI config.
  - Auto-merge is enabled at the repo level.                                                       
  - Label `needs-review` exists for paused PRs.                                                    
  - Run one full cycle now, then wait and run again. /loop handles the timing.                     
                                                                                                   

  PER-CYCLE BEHAVIOR                                              
                                                                                                   
  Step 1 — Discover:                                              
  Run: gh pr list --state open --json number,headRefName,isDraft,mergeable,statusCheckRollup,labels
   --jq '[.[] | select(.isDraft == false)]'                                                        

  Step 2 — Triage each PR into one of:                                                             
  - GREEN_CLEAN: CI passed, no conflicts, no `needs-review` label.
  - GREEN_CONFLICT: CI passed, merge conflict with main.                                           
  - RED: CI failed.                                                                                
  - PENDING: CI in progress. Skip.                                                                 
  - NO_CI: No CI on latest commit. Skip (it'll pick up next cycle).                                
  - AWAITING_REVIEW: Has `needs-review` label or open sub-PR assigned to me. Skip.                 
  - REVIEWED: Sub-PR previously assigned to me has merged. Resume.                                 
                                                                                                   

  Step 3 — Act:                                                                                    
                                                                                                   
  GREEN_CLEAN:                                                                                     
  - gh pr checks <n> to confirm.
  - gh pr merge <n> --squash --delete-branch                                                       
  - Log: "PR #<n>: merged."                                       

  GREEN_CONFLICT:                                                                                  
  - gh pr checkout <n>
  - git fetch origin main && git rebase origin/main                                                
  - Mechanical conflicts (different files, non-overlapping hunks, imports, formatting): resolve,
  run tests locally, git push --force-with-lease. Log "PR #<n>: rebased and pushed."               
  - Semantic conflicts (same logic, same API, same model touched both sides): git rebase --abort.
  Escalate via SUB-PR PROTOCOL with reason "semantic merge conflict."                              
                                                                  

  RED:                                                                                             
  - gh pr checks <n>, parse failure.                              
  - Classify:                                                                                      
    - TRIVIAL: clear bug in production code, isolated, confident fix, no test/CI/config changes
      needed.                                                                                          
    - SIGNIFICANT: anything else, OR a test appears wrong, OR you've already made one failed fix
      attempt on this PR.                                                                              
  - Trivial: gh pr checkout <n>, fix production code only, push. Max 2 trivial attempts per PR per
  fail mode — then escalate.                                                                       
  - Significant: escalate immediately via SUB-PR PROTOCOL.        
                                                                                                   

  PENDING / NO_CI / AWAITING_REVIEW: Log and skip.                                                 

  REVIEWED: Push an empty commit to retrigger CI: git commit --allow-empty -m "Retrigger CI after  
  sub-PR merge" && git push. Remove `needs-review` from parent: gh pr edit <n> --remove-label
  needs-review. Re-evaluate next cycle.                                                            
                                                                  
  SUB-PR PROTOCOL

  When a PR needs my judgment, do NOT modify the parent directly. Instead:                         

  1. gh pr checkout <parent-n>                                                                     
  2. git checkout -b <parent-branch>-fix-<short-desc>             
  3. Make the proposed changes (test changes, CI changes, refactors — whatever is needed; this is  
  the whole point).
  4. git push -u origin <sub-branch>                                                               
  5. Open sub-PR targeting parent's branch:                       
     gh pr create --base <parent-branch> --head <sub-branch> --title "Fix for #<parent-n>: <desc>" 
       --body "<see template>" --assignee @me --label needs-review                                      
  6. Enable auto-merge on the sub-PR: gh pr merge <sub-n> --auto --squash                          
  7. Add `needs-review` to PARENT: gh pr edit <parent-n> --add-label needs-review                  
  8. Comment on parent: gh pr comment <parent-n> --body "Paused. Needs your review on sub-PR       
  #<sub-n>."                                                                                       
  9. Move on.                                                                                      
                                                                                                   
  Sub-PR body template:                                           
  ---
  Parent PR: #<parent-n>
  Reason: <one line>

  Changes:
  - <bullets>

  Why escalated:                                                                                   
  - <1-2 lines>
                                                                                                   

  Failure detail (if applicable):                                 
  <max 10 lines log>

  What I need:
  - <specific question>
---

  HARD RULES (violating any stops the loop immediately)                                            

  - Never modify test files in the parent PR. Test changes go in sub-PRs only.                     
  - Never disable, skip, comment out, or delete tests anywhere.   
  - Never lower coverage thresholds in the parent PR.
  - Never modify CI config or workflow files in the parent PR.                                     
  - Never force-push to main or bypass branch protection.
  - Never merge a parent PR with failing or pending CI.                                            
  - Production code only when fixing parent PRs directly.                                          
  - Sub-PRs are exempt from the above — that's the whole point of escalation. I'll review them.
                                                                                                   

  CONFLICT RULES                                                                                   

  Mechanical (resolve): different files, non-overlapping hunks, imports, formatting, different     
  methods in same file.                                           
  Semantic (escalate): same function/method modified, same API signature changed differently, same
  model/schema touched, logic depending on the other side's behavior.                              

  When in doubt: it's semantic. Escalate.                                                          
                                                                  
  STOP CONDITIONS (end the loop, ping me)                                                          

  - Hard rule would be violated.                                                                   
  - External service failing (GitHub API, signing, registry, network).
  - Sub-PR can't be opened (auth/protection issue).
  - gh CLI auth failure.                                                                           
  - I tell you to stop.
                                                                                                   

  PAUSE CONDITIONS (skip PR, continue loop)                       

  - CI in progress.
  - Draft PR.
  - `needs-review` label.
  - PR pushed to <2 minutes ago.                                                                   
  - Sub-PR for this PR is open.
                                                                                                   

  REPORTING                                                       

  Sub-PR body + parent comment ARE the report. No extra ping needed for escalations — I see them   
  via GitHub notifications.
                                                                                                   
  If you stop the loop entirely:                                  
  LOOP STOPPED: <one-line reason>
  Last action: <what you did>
  Detail: <max 10 lines>                                                                           
  What I need: <decision>
                                                                                                   
  Be terse. No preamble. No apologies. No process narration.      

  CYCLE SUMMARY (end of each cycle)                                                                

  Cycle <n> at <timestamp>.                                                                        
  Merged: <count>                                                 
  Pushed fixes: <count>
  Sub-PRs opened: <count>
  Awaiting my review: <count>
  Skipped: <count>
                                                                                                   
  START NOW.
