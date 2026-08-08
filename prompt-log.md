*Stage 2 spec — 
draft/fix iteration Prompt: "Draft a Stage 2 FX hedge spec from my Stage 1 memo and scenario data, following the required sections and named-range contract." Gap found: the AI's first draft (before the real template-spec.md was available) set K_PUT to a rounded placeholder (1.1500) instead of the exact spot value, netted option premiums at face value instead of compounding them forward per the actual template's FV_PREM_PUT formula, and added a collar structure the template doesn't call for on the receivable side. Fix: reset K_PUT to the unrounded spot value with an explicit Stage-4 reconfirmation rule, switched to the template's compounded-premium formula (DF_USD-based), and removed the collar in favor of the template's put-only receivable hedge.

*Stage 3 build — 
AI-assisted, spec-driven
Prompt: "Build the workbook specified in my Stage 2 spec, following the Stage 3 build contract exactly (10 named ranges, formulas-only, cover/legend/inputs/per-hedge tabs, sensitivity + chart, validation checks live)."
Iteration: audit found table headers shared the same gray fill as named outputs, diluting the Legend/Key color contract — fixed by introducing a separate header color and reserving gray strictly for true outputs. Confirmed via a live recalculation test that the sensitivity grid updates correctly when inputs change.
