<!-- What & why: -->


## ⚠️ Two-repo pipeline checklist — merging THIS PR does NOT update docs.publora.com

The live site is built from [Co-Actor/post-actor](https://github.com/Co-Actor/post-actor), under `sites/docs.publora.com`. After merging here, synchronize and deploy that GitHub repository. The older GitLab mirror is not the current deployment source:

- [ ] Every changed file copied byte-identically to `post-actor/sites/docs.publora.com/content/…`
- [ ] If `schema/openapi.yaml` changed: `npm run openapi:generate-json` re-run (never hand-edit `openapi.json`)
- [ ] Offline gates pass in the site dir: `npm run docs:check-all`
- [ ] GitHub PR in `Co-Actor/post-actor` opened **and merged**
- [ ] Site deployed after the site PR merge (`npm run pages:build` + `wrangler deploy`)
- [ ] Edge cache purged for every changed URL (page + `/raw/<slug>` + `<slug>.md`; `llms.txt`/`llms-full.txt` if titles/descriptions changed) — the cache survives deploys
- [ ] Live spot-check: `curl https://docs.publora.com/raw/<slug>` shows the new content

Skipping these leaves the live site contradicting this repo (it has happened).
