# DAVID v6.0 — Welcome Dashboard

Render this widget when DAVID is called with **no code, no file, no error attached** (Mode 1).

Detect user language first. Then output the language-appropriate one-liner, followed by this widget:

- **EN**: `⚡ DAVID v6.0 — drop your code. I'll find what's wrong before you ship it.`
- **ID**: `⚡ DAVID v6.0 — drop kode kamu. Gw temukan masalahnya sebelum lo ship.`

---

```html
<style>
  .qs{display:flex;align-items:flex-start;gap:8px;width:100%;text-align:left;padding:10px 12px;border:0.5px solid var(--color-border-tertiary);border-radius:var(--border-radius-md);background:var(--color-background-primary);color:var(--color-text-primary);font-size:13px;cursor:pointer;transition:background 0.15s;margin-bottom:6px}
  .qs:hover{background:var(--color-background-secondary)}
  .qs .ic{font-size:15px;width:22px;flex-shrink:0;margin-top:1px}
  .qs .sub{color:var(--color-text-secondary);font-size:11px;margin-top:2px;line-height:1.4}
  .badge{display:inline-flex;align-items:center;font-size:11px;padding:2px 7px;border-radius:20px;background:var(--color-background-secondary);border:0.5px solid var(--color-border-tertiary);color:var(--color-text-secondary);margin:2px 1px}
  .sl{font-size:11px;font-weight:500;color:var(--color-text-tertiary);letter-spacing:.06em;text-transform:uppercase;margin:14px 0 7px}
  hr.d{border:none;border-top:0.5px solid var(--color-border-tertiary);margin:14px 0}
</style>
<div style="padding:2px 0 12px">
  <div style="display:flex;gap:14px;align-items:flex-start;margin-bottom:16px">
    <div style="width:44px;height:44px;border-radius:50%;background:var(--color-background-secondary);border:0.5px solid var(--color-border-secondary);display:flex;align-items:center;justify-content:center;font-size:20px;flex-shrink:0">🤖</div>
    <div>
      <div style="font-size:15px;font-weight:500">DAVID v6.0</div>
      <div id="dv-sub" style="font-size:13px;color:var(--color-text-secondary);margin-top:3px;line-height:1.5"></div>
    </div>
  </div>
  <div style="display:grid;grid-template-columns:repeat(4,1fr);gap:8px;margin-bottom:4px">
    <div style="background:var(--color-background-secondary);border-radius:var(--border-radius-md);padding:10px 8px;text-align:center"><div style="font-size:19px;font-weight:500">40+</div><div style="font-size:11px;color:var(--color-text-secondary);margin-top:1px">scanners</div></div>
    <div style="background:var(--color-background-secondary);border-radius:var(--border-radius-md);padding:10px 8px;text-align:center"><div style="font-size:19px;font-weight:500">43</div><div style="font-size:11px;color:var(--color-text-secondary);margin-top:1px">ref files</div></div>
    <div style="background:var(--color-background-secondary);border-radius:var(--border-radius-md);padding:10px 8px;text-align:center"><div style="font-size:19px;font-weight:500">5x</div><div style="font-size:11px;color:var(--color-text-secondary);margin-top:1px">fix loops</div></div>
    <div style="background:var(--color-background-secondary);border-radius:var(--border-radius-md);padding:10px 8px;text-align:center"><div style="font-size:19px;font-weight:500">0</div><div style="font-size:11px;color:var(--color-text-secondary);margin-top:1px">silent del.</div></div>
  </div>
  <hr class="d">
  <div id="dv-cta" class="sl"></div>
  <div id="dv-btns"></div>
  <hr class="d">
  <div id="dv-footer" class="sl"></div>
  <div style="line-height:2.1">
    <span class="badge">🔍 Bug 5-pass</span><span class="badge">🔐 Security</span><span class="badge">📊 Perf</span><span class="badge">🎨 UX x14</span><span class="badge">🚀 Enhance x12</span><span class="badge">♿ WCAG 2.2</span><span class="badge">🔷 TypeScript</span><span class="badge">🧪 Test Quality</span><span class="badge">🔏 GDPR</span><span class="badge">🛡️ Resilience</span><span class="badge">🗄️ DB Schema</span><span class="badge">🔍 SEO</span><span class="badge">📡 Observability</span><span class="badge">📦 Bundle</span><span class="badge">🧩 State Mgmt</span><span class="badge">📱 PWA</span><span class="badge">⚙️ Env Config</span><span class="badge">🚩 Feature Flags</span><span class="badge">📋 API Contract</span><span class="badge">💀 Dead Code</span><span class="badge">📐 CC Score</span><span class="badge">🌍 i18n</span><span class="badge">🗺️ Debt Map</span><span class="badge">⬆️ Upgrade Adv</span><span class="badge">📝 PR Desc</span><span class="badge">🏥 Health Score</span><span class="badge">🔀 Git Diff</span><span class="badge">🎯 Vibe Code</span><span class="badge">+ more</span>
  </div>
  <hr class="d">
  <div id="dv-hint" style="font-size:12px;color:var(--color-text-tertiary);line-height:1.5"></div>
</div>
<script>
(function(){
  var lang = (navigator.language||'en').toLowerCase().startsWith('id') ? 'id' : 'en';
  var t = {
    en: {
      sub: 'Principal Engineer. 40+ scanners armed. I find bugs, fix them, enhance what works, test, document — then loop until zero issues remain.',
      cta: 'What do you need? Pick one — then attach your code',
      footer: '40+ scanners active · bilingual (EN/ID)',
      hint: '💡 Paste code + describe the problem in one message — DAVID auto-detects mode. Write in EN or ID — DAVID follows your language.',
      btns: [
        {ic:'🔁',t:'Full scan — all scanners',s:'Bug + Security + Perf + UX + Types + SEO + all — health score at end',p:'David, here is my code — please full scan: bug, security, performance, UX, types, and give health score at the end.'},
        {ic:'🐛',t:'Debug & fix error / crash',s:'Paste error message, stack trace, or describe the bug',p:'David, I have this error — please debug and fix completely:'},
        {ic:'🔐',t:'Security + Privacy audit',s:'OWASP Top 10, secrets detection, GDPR Art.17, supply chain risks',p:'David, deep security audit: OWASP, secrets, GDPR/privacy, supply chain, input sanitization.'},
        {ic:'🎨',t:'UI/UX + Accessibility audit',s:'14 sub-scanners: visual, states, forms, mobile, WCAG 2.2, delight',p:'David, audit this frontend UI/UX: visual design, interaction states, form UX, mobile touch, dark mode, accessibility, microcopy, emotional design.'},
        {ic:'🤝',t:'Pre-PR code review',s:'Code quality, SOLID, CC score, dead code, TypeScript, upgrade advice',p:'David, pre-PR code review: quality, architecture, SOLID, type safety, dead code, complexity score, upgrade suggestions.'},
        {ic:'🧪',t:'Test quality audit + generate',s:'Flaky detection, weak assertions, coverage gaps, edge case generation',p:'David, generate comprehensive unit tests + audit existing tests: weak assertions, flaky tests, missing edge cases.'},
        {ic:'📊',t:'Performance + Resilience audit',s:'N+1, Big-O, bundle, memory, timeouts, circuit breakers, DB indexes',p:'David, performance profiling: N+1 queries, Big-O, bundle size, memory leaks, resilience patterns, DB schema index coverage.'},
        {ic:'🗺️',t:'Tech debt map + refactor plan',s:'TODO inventory, CC scores, debt ranking, step-by-step migration',p:'David, tech debt heatmap + refactoring plan: inventory TODO/FIXME, rank the most problematic files, build a safe migration roadmap.'},
        {ic:'🎯',t:'Vibe Code audit (AI-generated)',s:'TODO debris, fake APIs, console.log, any epidemic, duplicate boilerplate',p:'David, this code is from AI/Copilot — scan vibe code patterns: TODO debris, hallucinated APIs, console.log, any epidemic, missing edge cases, copy-paste duplicates.'},
        {ic:'🚀',t:'Enhance UI/UX (upgrade mode)',s:'Loading → skeleton, empty state → illustrated, microinteractions, UX maturity score',p:'David, enhance this UI/UX — not just fix what is broken, but upgrade what already works: loading states, empty states, microinteractions, form UX, navigation, visual hierarchy. Give UX maturity score before and after.'}
      ]
    },
    id: {
      sub: 'Principal Engineer. 40+ scanner siap. Gw temukan bug, fix, enhance yang sudah ada, test, dokumentasi — lalu loop sampai zero issues.',
      cta: 'Mau ngapain? Klik salah satu — terus lampirkan kode kamu',
      footer: '40+ scanner aktif · bilingual (EN/ID)',
      hint: '💡 Paste kode + describe masalah dalam satu pesan — DAVID auto-detect mode. Ketik EN atau ID — DAVID ikuti bahasamu.',
      btns: [
        {ic:'🔁',t:'Full scan semua scanner',s:'Bug + Security + Perf + UX + Types + SEO + semua — health score di akhir',p:'David, ini kode saya — tolong full scan: bug, security, performance, UX, types, dan kasih health score di akhir.'},
        {ic:'🐛',t:'Debug & fix error / crash',s:'Paste error message, stack trace, atau describe bug-nya',p:'David, ada error ini — tolong debug dan fix sampai tuntas:'},
        {ic:'🔐',t:'Security + Privacy audit',s:'OWASP Top 10, secrets detection, GDPR Art.17, supply chain risks',p:'David, security audit mendalam: OWASP, secrets, GDPR/privacy, supply chain, input sanitization.'},
        {ic:'🎨',t:'UI/UX + Accessibility audit',s:'14 sub-scanner: visual, states, forms, mobile, WCAG 2.2, delight',p:'David, audit UI/UX frontend ini: visual design, interaction states, form UX, mobile touch, dark mode, accessibility, microcopy, emotional design.'},
        {ic:'🤝',t:'Pre-PR code review',s:'Code quality, SOLID, CC score, dead code, TypeScript, upgrade advice',p:'David, pre-PR code review: quality, architecture, SOLID, type safety, dead code, complexity score, upgrade suggestions.'},
        {ic:'🧪',t:'Test quality audit + generate',s:'Flaky detection, weak assertions, coverage gaps, edge case generation',p:'David, generate unit tests comprehensive + audit existing tests: weak assertions, flaky tests, missing edge cases.'},
        {ic:'📊',t:'Performance + Resilience audit',s:'N+1, Big-O, bundle, memory, timeouts, circuit breakers, DB indexes',p:'David, performance profiling: N+1 queries, Big-O, bundle size, memory leaks, resilience patterns, DB schema index coverage.'},
        {ic:'🗺️',t:'Tech debt map + refactor plan',s:'TODO inventory, CC scores, debt ranking, step-by-step migration',p:'David, tech debt heatmap + refactoring plan: inventarisir TODO/FIXME, rank file paling bermasalah, buat langkah migrasi yang safe.'},
        {ic:'🎯',t:'Vibe Code audit (AI-generated code)',s:'TODO debris, fake APIs, console.log, any epidemic, duplicate boilerplate',p:'David, ini kode dari AI/Copilot — scan vibe code patterns: TODO debris, hallucinated APIs, console.log, any epidemic, missing edge cases, copy-paste duplicates.'},
        {ic:'🚀',t:'Enhance UI/UX (upgrade mode)',s:'Loading → skeleton, empty state → illustrated, microinteractions, UX maturity score',p:'David, enhance UI/UX ini — jangan cuma fix yang salah, tapi upgrade yang sudah ada: loading states, empty states, microinteractions, form UX, navigasi, visual hierarchy. Kasih UX maturity score sebelum dan sesudah.'}
      ]
    }
  };
  var L = t[lang];
  document.getElementById('dv-sub').textContent = L.sub;
  document.getElementById('dv-cta').textContent = L.cta;
  document.getElementById('dv-footer').textContent = L.footer;
  document.getElementById('dv-hint').textContent = L.hint;
  var html = L.btns.map(function(b){
    return '<button class="qs" onclick="sendPrompt(\''+b.p.replace(/'/g,"\\'")+'\'"><span class="ic">'+b.ic+'</span><div><div style="font-weight:500">'+b.t+'</div><div class="sub">'+b.s+'</div></div></button>';
  }).join('');
  document.getElementById('dv-btns').innerHTML = html;
})();
</script>
```
