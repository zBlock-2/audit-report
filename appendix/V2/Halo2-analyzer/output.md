# v2

Choose the mode of analysis for your circuit.
1. Unused Gates
2. Unused Columns
3. Unconstrained Cells
4. Underconstrained Circuit
1
Finished analysis: 0 unused gates found.

Choose the mode of analysis for your circuit.
1. Unused Gates
2. Unused Columns
3. Unconstrained Cells
4. Underconstrained Circuit
2
Finished analysis: 5 unused columns found.
```
unused column: Column { index: 2, column_type: Advice }
unused column: Column { index: 3, column_type: Advice }
unused column: Column { index: 4, column_type: Advice }
unused column: Column { index: 5, column_type: Advice }
unused column: Column { index: 1, column_type: Advice }
```

Choose the mode of analysis for your circuit.
1. Unused Gates
2. Unused Columns
3. Unconstrained Cells
4. Underconstrained Circuit
3
Finished analysis: 12 unconstrained cells found.
```
unconstrained cell in "assign entries to the table" region: Column { index: 1, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "assign entries to the table" region: Column { index: 0, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "assign entries to the table" region: Column { index: 0, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "assign entries to the table" region: Column { index: 1, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "Perform range check on balance 0 of user 0" region: Column { index: 2, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "Perform range check on balance 0 of user 0" region: Column { index: 5, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "Perform range check on balance 0 of user 0" region: Column { index: 3, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "Perform range check on balance 0 of user 0" region: Column { index: 4, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "Perform range check on balance 0 of user 1" region: Column { index: 2, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "Perform range check on balance 0 of user 1" region: Column { index: 3, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "Perform range check on balance 0 of user 1" region: Column { index: 4, column_type: Advice } (rotation: 0) -- very likely a bug.
unconstrained cell in "Perform range check on balance 0 of user 1" region: Column { index: 5, column_type: Advice } (rotation: 0) -- very likely a bug.
```


