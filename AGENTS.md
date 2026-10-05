<!-- LOVABLE:BEGIN -->
> [!IMPORTANT]
> This project is connected to [Lovable](https://lovable.dev). Avoid rewriting
> published git history — force pushing, or rebasing/amending/squashing commits
> that are already pushed — as it rewrites history on Lovable's side and the
> user will likely lose their project history.
>
> Commits you push to the connected branch sync back to Lovable and show up in
> the editor, so keep the branch in a working state.
<!-- LOVABLE:END -->

## Architecture rules
- Persistent storage is a user-deployed Google Apps Script web app over Google Sheets (`public/apps-script/Code.gs`); the client calls it with text/plain POST + APP_TOKEN. Why: user requires Sheets as the backend and Apps Script cannot answer CORS preflights.
- Money is stored as integer cents and dates as America/Lima wall-clock strings (`YYYY-MM-DDTHH:mm`). Why: avoids rounding and timezone drift between browser, Script and Sheets.
- Balances, debts, budgets and goal totals are derived from records (never stored). Why: edits/deletes recompute everything without double-applying.
- Scheduled items (auto recurring, card insurance) use deterministic ids and "insert-if-absent" ops; logic in `src/lib/calc.ts` is mirrored in Code.gs. Why: client and daily trigger can both run without duplicates.
- Every write goes through `useSubmit`/`mutate` with a stable requestId; the server dedupes requests under a script lock. Why: prevents duplicates from double clicks and retries.
