# Prefabs

Browse, search and filter all available game prefabs. Use the **category sidebar** to narrow down by type, the **search bar** to find by name, ID or translation, and the **language switcher** to see in-game text in your preferred language.

<style>
:root {
  --content-max-width: 70%;
}

#prefabs-app {
  display: flex;
  gap: 16px;
  align-items: flex-start;
  margin-top: 20px;
  width: 100%;
}

/* ── Category sidebar ── */
#prefabs-categories {
  flex: 0 0 190px;
  background: rgba(255,255,255,0.04);
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: 6px;
  overflow: hidden;
  position: sticky;
  top: 80px;
  max-height: calc(100vh - 120px);
  display: flex;
  flex-direction: column;
}

#prefabs-cat-header {
  padding: 10px 12px;
  font-size: 0.72em;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: rgba(255,255,255,0.35);
  border-bottom: 1px solid rgba(255,255,255,0.08);
  flex-shrink: 0;
}

#prefabs-cat-search-wrap {
  padding: 8px 8px;
  border-bottom: 1px solid rgba(255,255,255,0.08);
  flex-shrink: 0;
}

#prefabs-cat-search {
  width: 100%;
  box-sizing: border-box;
  background: rgba(255,255,255,0.06);
  border: 1px solid rgba(255,255,255,0.1);
  border-radius: 4px;
  color: inherit;
  padding: 5px 8px;
  font-size: 0.8em;
  outline: none;
}

#prefabs-cat-search:focus {
  border-color: #c23030;
}

#prefabs-cat-list {
  overflow-y: auto;
  flex: 1;
  padding: 5px;
}

.prefab-cat-btn {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  padding: 5px 8px;
  background: none;
  border: none;
  border-radius: 4px;
  color: inherit;
  cursor: pointer;
  font-size: 0.8em;
  text-align: left;
  margin-bottom: 2px;
  gap: 4px;
  transition: background 0.12s;
}

.prefab-cat-btn:hover {
  background: rgba(255,255,255,0.07);
}

.prefab-cat-btn.active {
  background: #c23030;
  color: #fff;
}

.prefab-cat-name {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  flex: 1;
}

.prefab-cat-count {
  flex-shrink: 0;
  background: rgba(0,0,0,0.25);
  border-radius: 10px;
  padding: 1px 5px;
  font-size: 0.82em;
  font-variant-numeric: tabular-nums;
}

.prefab-cat-btn.active .prefab-cat-count {
  background: rgba(0,0,0,0.3);
}

/* ── Main content ── */
#prefabs-content {
  flex: 1 1 0;
  min-width: 0;
  width: 0;
  overflow: hidden;
}

#prefabs-controls {
  display: flex;
  gap: 10px;
  margin-bottom: 14px;
  flex-wrap: wrap;
}

#prefabs-search {
  flex: 1;
  min-width: 200px;
  padding: 8px 12px;
  background: rgba(255,255,255,0.06);
  border: 1px solid rgba(255,255,255,0.1);
  border-radius: 4px;
  color: inherit;
  font-size: 0.88em;
  outline: none;
}

#prefabs-search:focus {
  border-color: #c23030;
}

#prefabs-lang {
  padding: 8px 12px;
  background: rgba(255,255,255,0.06);
  border: 1px solid rgba(255,255,255,0.1);
  border-radius: 4px;
  color: inherit;
  font-size: 0.88em;
  outline: none;
  cursor: pointer;
  color-scheme: dark;
}

#prefabs-lang option {
  background: #1e1e1e;
  color: #e0e0e0;
}

#prefabs-lang:focus {
  border-color: #c23030;
}

/* ── Grid table ── */
#prefabs-grid {
  width: 100%;
  font-size: 0.95em;
}

#prefabs-grid-head,
.prefab-row {
  display: grid;
  grid-template-columns: minmax(0, 1fr) 170px minmax(0, 1fr);
  align-items: center;
  width: 100%;
}

#prefabs-grid-head {
  border-bottom: 2px solid #c23030;
  padding: 9px 0;
}

#prefabs-grid-head > div {
  padding: 0 20px;
  font-weight: 600;
  font-size: 0.82em;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  color: rgba(255,255,255,0.55);
  white-space: nowrap;
}

.prefab-row {
  border-bottom: 1px solid rgba(255,255,255,0.05);
  transition: background 0.1s;
}

.prefab-row:nth-child(odd)  { background: rgba(0,0,0,0.18); }
.prefab-row:nth-child(even) { background: rgba(255,255,255,0.04); }
.prefab-row:hover           { background: rgba(255,255,255,0.08); }

.prefab-row > div {
  padding: 14px 20px;
}

.prefab-col-name {
  font-family: monospace;
  font-size: 0.9em;
  font-weight: 700;
  word-break: break-word;
  overflow-wrap: break-word;
  overflow: hidden;
}

.prefab-col-id {
  font-family: monospace;
  text-align: right;
  color: #c23030;
  white-space: nowrap;
  overflow: visible;
  position: relative;
  z-index: 10;
}

.prefab-col-id span {
  cursor: pointer;
  position: relative;
  border-bottom: 1px dashed rgba(194,48,48,0.5);
  transition: color 0.15s;
}

.prefab-col-id span:hover {
  color: #e05050;
}

.prefab-col-id span::after {
  content: 'Click to copy';
  position: absolute;
  bottom: 120%;
  left: 50%;
  transform: translateX(-50%);
  background: #333;
  color: #fff;
  font-size: 0.75em;
  padding: 3px 8px;
  border-radius: 4px;
  white-space: nowrap;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.15s;
}

.prefab-col-id span:hover::after {
  opacity: 1;
}

.prefab-col-id span.copied {
  color: #4caf50 !important;
  border-color: #4caf50;
}

.prefab-col-id span.copied::after {
  content: 'Copied!';
  opacity: 1;
  background: #2e7d32;
}

.prefab-col-trans {
  color: rgba(255,255,255,0.8);
}

.prefab-no-results {
  text-align: center;
  padding: 32px;
  opacity: 0.45;
  font-style: italic;
}

/* ── Pagination ── */
#prefabs-pagination {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 16px;
  margin-top: 16px;
  padding: 14px 0 4px;
  border-top: 1px solid rgba(255,255,255,0.08);
}

#prefabs-pagination button {
  padding: 7px 18px;
  background: rgba(255,255,255,0.06);
  border: 1px solid rgba(255,255,255,0.1);
  border-radius: 4px;
  color: inherit;
  cursor: pointer;
  font-size: 0.84em;
  transition: background 0.12s, border-color 0.12s;
}

#prefabs-pagination button:hover:not(:disabled) {
  background: rgba(194,48,48,0.18);
  border-color: #c23030;
}

#prefabs-pagination button:disabled {
  opacity: 0.28;
  cursor: default;
}

#prefabs-page-info {
  font-size: 0.82em;
  color: rgba(255,255,255,0.4);
  text-align: center;
  min-width: 210px;
}

/* ── Responsive ── */
@media (max-width: 1249px) {
  :root { --content-max-width: 95%; }
}

@media (max-width: 720px) {
  #prefabs-app { flex-direction: column; }
  #prefabs-categories {
    flex: none;
    width: 100%;
    position: static;
    max-height: 220px;
  }
}
</style>

<div id="prefabs-app">

  <aside id="prefabs-categories">
    <div id="prefabs-cat-header">Categories</div>
    <div id="prefabs-cat-search-wrap">
      <input id="prefabs-cat-search" type="text" placeholder="Filter categories…" autocomplete="off" />
    </div>
    <div id="prefabs-cat-list">
      <div style="padding:12px;opacity:0.4;font-size:0.82em">Loading…</div>
    </div>
  </aside>

  <section id="prefabs-content">
    <div id="prefabs-controls">
      <input id="prefabs-search" type="text" placeholder="Search… e.g. weapon, chest +t06 -broken" autocomplete="off" />
      <select id="prefabs-lang"><option>Loading…</option></select>
    </div>
    <div id="prefabs-grid">
      <div id="prefabs-grid-head">
        <div>Prefab Name</div>
        <div style="text-align:right">Prefab ID</div>
        <div>Translation</div>
      </div>
      <div id="prefabs-grid-body">
        <div class="prefab-no-results">Loading prefabs…</div>
      </div>
    </div>
    <div id="prefabs-pagination">
      <button id="prefabs-prev" disabled>← Previous</button>
      <span id="prefabs-page-info">–</span>
      <button id="prefabs-next" disabled>Next →</button>
    </div>
  </section>

</div>

<script>
(function () {
  var PAGE_SIZE = 50;
  var allData = [];
  var filtered = [];
  var currentPage = 1;
  var currentLang = 'English';
  var currentCategory = '__all__';
  var searchTerm = '';
  var catSearchTerm = '';
  var translationsData = null;
  var categories = [];
  var catCounts = {};

  function $id(id) { return document.getElementById(id); }

  function esc(str) {
    if (str == null) return '';
    return String(str)
      .replace(/&/g, '&amp;')
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;');
  }

  function getCategory(name) {
    var i = name.indexOf('_');
    return i > 0 ? name.substring(0, i) : '(Other)';
  }

  function getTranslation(d) {
    if (!d.guid || !translationsData) return 'No name';
    var entry = translationsData[d.guid];
    if (!entry) return 'No name';
    var t = entry[currentLang];
    if (t == null || String(t).trim() === '') return 'No name';
    return t;
  }

  // ──────────────────────────────────────────────
  // Load data
  // ──────────────────────────────────────────────
  var base = window.location.pathname.replace(/\/index\.html$/, '').replace(/\/$/, '');

  Promise.all([
    fetch(base + '/AllPrefabs.json').then(function (r) { return r.json(); }),
    fetch(base + '/PrefabToGuidMap.json').then(function (r) { return r.json(); }),
    fetch(base + '/GameTranslations.json').then(function (r) { return r.json(); })
  ]).then(function (res) {
    var prefabs = res[0];
    var guidMap = res[1];
    translationsData = res[2];

    // Build language list
    var firstKey = Object.keys(translationsData)[0];
    var langs = firstKey ? Object.keys(translationsData[firstKey]).sort() : ['English'];
    currentLang = langs.indexOf('English') !== -1 ? 'English' : langs[0];

    var sel = $id('prefabs-lang');
    sel.innerHTML = '';
    langs.forEach(function (l) {
      var o = document.createElement('option');
      o.value = l;
      o.textContent = l;
      if (l === currentLang) o.selected = true;
      sel.appendChild(o);
    });

    // Build rows
    allData = Object.keys(prefabs).map(function (name) {
      var id = prefabs[name];
      return {
        name: name,
        id: id,
        guid: guidMap[String(id)] || null,
        category: getCategory(name)
      };
    });

    // Category counts
    allData.forEach(function (d) {
      catCounts[d.category] = (catCounts[d.category] || 0) + 1;
    });
    categories = Object.keys(catCounts).sort(function (a, b) { return a.localeCompare(b); });

    renderCategories();
    applyFilters();

  }).catch(function (err) {
    $id('prefabs-grid-body').innerHTML =
      '<div style="text-align:center;color:#e55;padding:24px">Failed to load data: ' + esc(err.message) + '</div>';
  });

  // ──────────────────────────────────────────────
  // Render categories
  // ──────────────────────────────────────────────
  function renderCategories() {
    var term = catSearchTerm.toLowerCase();
    var list = $id('prefabs-cat-list');
    list.innerHTML = '';

    // "All" button
    var allBtn = document.createElement('button');
    allBtn.className = 'prefab-cat-btn' + (currentCategory === '__all__' ? ' active' : '');
    allBtn.innerHTML =
      '<span class="prefab-cat-name">All</span>' +
      '<span class="prefab-cat-count">' + allData.length.toLocaleString() + '</span>';
    allBtn.addEventListener('click', function () { setCategory('__all__'); });
    list.appendChild(allBtn);

    var visible = term
      ? categories.filter(function (c) { return c.toLowerCase().indexOf(term) !== -1; })
      : categories;

    visible.forEach(function (cat) {
      var btn = document.createElement('button');
      btn.className = 'prefab-cat-btn' + (currentCategory === cat ? ' active' : '');
      btn.innerHTML =
        '<span class="prefab-cat-name">' + esc(cat) + '</span>' +
        '<span class="prefab-cat-count">' + (catCounts[cat] || 0) + '</span>';
      btn.addEventListener('click', function () { setCategory(cat); });
      list.appendChild(btn);
    });
  }

  function setCategory(cat) {
    currentCategory = cat;
    currentPage = 1;
    renderCategories();
    applyFilters();
  }

  // ──────────────────────────────────────────────
  // Filter + rank + render
  // ──────────────────────────────────────────────
  function scoreItem(d, terms) {
    var name  = d.name.toLowerCase();
    var id    = String(d.id);
    var trans = getTranslation(d).toLowerCase();
    var total = 0;

    for (var i = 0; i < terms.length; i++) {
      var t = terms[i];
      var s = 0;
      var matched = false;

      // Name: exact > starts-with > word-boundary > contains
      if (name === t)                   { s += 100; matched = true; }
      else if (name.indexOf(t) === 0)   { s +=  70; matched = true; }
      else if (name.indexOf('_' + t) !== -1) { s += 55; matched = true; }
      else if (name.indexOf(t) !== -1)  { s +=  40; matched = true; }

      // ID: exact > contains
      if (id === t)                     { s +=  90; matched = true; }
      else if (id.indexOf(t) !== -1)    { s +=  30; matched = true; }

      // Translation: exact > starts-with > contains
      if (trans === t)                  { s +=  80; matched = true; }
      else if (trans.indexOf(t) === 0)  { s +=  50; matched = true; }
      else if (trans.indexOf(t) !== -1) { s +=  20; matched = true; }

      if (!matched) return -1;  // term not found at all → exclude item
      total += s;
    }
    return total;
  }

  // Parse "weapon, chest +item +t06 -broken" into groups/required/excluded
  function parseSearch(raw) {
    var globalRequired = [];
    var globalExcluded = [];

    function isNegativeNumber(tok) {
      return tok[0] === '-' && tok.length > 1 && /^\d+$/.test(tok.slice(1));
    }

    // Collect all +/- tokens from the entire input
    raw.split(/\s+/).forEach(function (tok) {
      if (tok.length < 2) return;
      if (tok[0] === '+') globalRequired.push(tok.slice(1).toLowerCase());
      else if (tok[0] === '-' && !isNegativeNumber(tok)) globalExcluded.push(tok.slice(1).toLowerCase());
    });

    // Split by comma into groups; strip +/- modifier tokens but keep negative numbers
    var groups = raw.split(',').map(function (g) {
      return g.trim().split(/\s+/).filter(function (t) {
        if (!t) return false;
        if (t[0] === '+') return false;
        if (t[0] === '-' && !isNegativeNumber(t)) return false;
        return true;
      }).map(function (t) { return t.toLowerCase(); });
    }).filter(function (g) { return g.length > 0; });

    return { groups: groups, required: globalRequired, excluded: globalExcluded };
  }

  function applyFilters() {
    var parsed = parseSearch(searchTerm);
    var groups   = parsed.groups;
    var required = parsed.required;
    var excluded = parsed.excluded;

    var hasQuery = groups.length > 0 || required.length > 0 || excluded.length > 0;

    if (!hasQuery) {
      filtered = allData.filter(function (d) {
        return currentCategory === '__all__' || d.category === currentCategory;
      });
      renderTable();
      renderPagination();
      return;
    }

    // If only +/- modifiers and no comma groups, treat required as a single group
    var effectiveGroups = groups.length > 0 ? groups : (required.length > 0 ? [[]] : []);

    var scored = [];
    allData.forEach(function (d) {
      if (currentCategory !== '__all__' && d.category !== currentCategory) return;

      var name  = d.name.toLowerCase();
      var id    = String(d.id);
      var trans = getTranslation(d).toLowerCase();

      // Apply global exclusions first
      for (var ei = 0; ei < excluded.length; ei++) {
        var ex = excluded[ei];
        if (name.indexOf(ex) !== -1 || id.indexOf(ex) !== -1 || trans.indexOf(ex) !== -1) return;
      }

      // Must match at least one group (group terms + required)
      var bestScore = -1;
      for (var gi = 0; gi < effectiveGroups.length; gi++) {
        var groupTerms = effectiveGroups[gi].concat(required);
        if (groupTerms.length === 0) { bestScore = Math.max(bestScore, 0); continue; }
        var s = scoreItem(d, groupTerms);
        if (s > bestScore) bestScore = s;
      }

      if (bestScore >= 0) scored.push({ d: d, score: bestScore });
    });

    scored.sort(function (a, b) { return b.score - a.score; });
    filtered = scored.map(function (s) { return s.d; });

    renderTable();
    renderPagination();
  }

  function renderTable() {
    var start = (currentPage - 1) * PAGE_SIZE;
    var rows = filtered.slice(start, start + PAGE_SIZE);
    var body = $id('prefabs-grid-body');
    if (!rows.length) {
      body.innerHTML = '<div class="prefab-no-results">No prefabs found</div>';
      return;
    }
    body.innerHTML = rows.map(function (d) {
      return '<div class="prefab-row">' +
        '<div class="prefab-col-name">' + esc(d.name) + '</div>' +
        '<div class="prefab-col-id"><span data-id="' + d.id + '">' + d.id + '</span></div>' +
        '<div class="prefab-col-trans">' + esc(getTranslation(d)) + '</div>' +
        '</div>';
    }).join('');
  }

  function renderPagination() {
    var total = Math.max(1, Math.ceil(filtered.length / PAGE_SIZE));
    $id('prefabs-page-info').textContent =
      'Page ' + currentPage + ' of ' + total + '  ·  ' + filtered.length.toLocaleString() + ' prefabs';
    $id('prefabs-prev').disabled = currentPage <= 1;
    $id('prefabs-next').disabled = currentPage >= total;
  }

  // ──────────────────────────────────────────────
  // Event listeners
  // ──────────────────────────────────────────────
  $id('prefabs-search').addEventListener('input', function (e) {
    searchTerm = e.target.value;
    currentPage = 1;
    applyFilters();
  });

  $id('prefabs-lang').addEventListener('change', function (e) {
    currentLang = e.target.value;
    applyFilters();
  });

  $id('prefabs-cat-search').addEventListener('input', function (e) {
    catSearchTerm = e.target.value;
    renderCategories();
  });

  $id('prefabs-grid-body').addEventListener('click', function (e) {
    var span = e.target.closest('.prefab-col-id span');
    if (!span) return;
    var id = span.getAttribute('data-id');
    navigator.clipboard.writeText(id).then(function () {
      span.classList.add('copied');
      setTimeout(function () { span.classList.remove('copied'); }, 1500);
    });
  });

  $id('prefabs-prev').addEventListener('click', function () {
    if (currentPage > 1) { currentPage--; renderTable(); renderPagination(); }
  });

  $id('prefabs-next').addEventListener('click', function () {
    var total = Math.ceil(filtered.length / PAGE_SIZE);
    if (currentPage < total) { currentPage++; renderTable(); renderPagination(); }
  });

}());
</script>

<script>
// ── Auto-close docsify sidebar on screens < 1250px ──
(function () {
  var autoClosed = false;

  function manageSidebar() {
    var sidebar = document.querySelector('.sidebar');
    var toggle  = document.querySelector('.sidebar-toggle');
    if (!sidebar || !toggle) return;
    var isSmall  = window.innerWidth < 1250;
    var isClosed = sidebar.classList.contains('close');
    if (isSmall && !isClosed) {
      toggle.click();
      autoClosed = true;
    } else if (!isSmall && isClosed && autoClosed) {
      toggle.click();
      autoClosed = false;
    }
  }

  manageSidebar();
  window.addEventListener('resize', manageSidebar);
}());
</script>
