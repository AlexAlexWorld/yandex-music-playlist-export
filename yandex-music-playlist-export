(() => {
  const tracks = new Map();
  let running = true;

  const CFG = {
    debounceMs: 120,
    settleMs: 10000,
    settleTickMs: 250
  };

  const safeText = el => (el && el.textContent ? el.textContent.trim() : '');

  const injectStyles = () => {
    const oldStyle = document.getElementById('ym-scraper-styles');
    if (oldStyle) oldStyle.remove();

    const style = document.createElement('style');
    style.id = 'ym-scraper-styles';
    style.textContent = `
      #ym-scraper {
        position: fixed;
        top: 24px;
        right: 24px;
        z-index: 999999;
        width: 320px;
        background: #0e0e0e;
        border: 1px solid #222;
        border-radius: 12px;
        padding: 18px;
        font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Courier New", monospace;
        font-size: 12px;
        color: #999;
        box-shadow: 0 8px 40px rgba(0,0,0,.6);
      }
      #ym-scraper .h {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 10px;
      }
      #ym-scraper .l {
        font-size: 10px;
        letter-spacing: .12em;
        text-transform: uppercase;
        color: #444;
      }
      #ym-scraper .dot {
        width: 7px;
        height: 7px;
        border-radius: 50%;
        background: #3b3b3b;
      }
      #ym-scraper .dot.a {
        background: #c8f560;
        box-shadow: 0 0 8px rgba(200,245,96,.5);
      }
      #ym-scraper .c {
        font-size: 34px;
        color: #f0f0f0;
        line-height: 1;
        margin: 6px 0;
      }
      #ym-scraper .s {
        font-size: 11px;
        color: #666;
        min-height: 14px;
        margin: 10px 0 12px;
      }
      #ym-scraper .m {
        font-size: 10px;
        color: #555;
        margin-bottom: 10px;
      }
      #ym-scraper button {
        width: 100%;
        padding: 9px 12px;
        border: 1px solid #2a2a2a;
        border-radius: 8px;
        background: transparent;
        color: #ccc;
        font: inherit;
        font-size: 11px;
        letter-spacing: .06em;
        cursor: pointer;
        text-align: left;
        margin-top: 8px;
      }
      #ym-scraper button:hover {
        background: #1a1a1a;
        border-color: #444;
        color: #fff;
      }
      #ym-scraper button.p {
        border-color: #c8f560;
        color: #c8f560;
      }
      #ym-scraper button.p:hover {
        background: rgba(200,245,96,.07);
      }
      #ym-scraper .f {
        margin-top: 12px;
        font-size: 10px;
        color: #2e2e2e;
      }
      #ym-scraper .f span {
        color: #c8f560;
      }
    `;
    document.head.appendChild(style);
  };

  const getPanel = () => {
    let panel = document.getElementById('ym-scraper');
    if (!panel) {
      panel = document.createElement('div');
      panel.id = 'ym-scraper';
      document.body.appendChild(panel);
    }
    return panel;
  };

  const parseRow = row => {
    let id = '';
    let title = '';
    let artists = '';
    let source = 'normal';

    const trackLink = row.querySelector('a[href*="/album/"][href*="/track/"]');
    if (trackLink) {
      const href = trackLink.getAttribute('href') || '';
      const m = href.match(/\/track\/(\d+)/);
      if (m) id = m[1];
    }

    const titleEl = row.querySelector('.Meta_title__GGBnH');
    title = safeText(titleEl);

    const versionEl = row.querySelector('.Meta_version__c2sHU');
    const version = safeText(versionEl);
    if (title && version) {
      title = `${title} ${version}`.replace(/\s+/g, ' ').trim();
    }

    const artistSpans = row.querySelectorAll('.Meta_artists__VnR52 .Meta_artistCaption__JESZi');
    artists = Array.from(artistSpans).map(safeText).filter(Boolean).join(', ');

    if (!title) {
      const img = row.querySelector('img[alt^="Трек "]');
      const alt = img?.getAttribute('alt') || '';
      const m = alt.match(/^Трек\s+(.+)$/);
      if (m) {
        title = m[1].trim();
        source = 'fallback';
      }
    }

    if (!artists) {
      const aria = row.getAttribute('aria-label') || '';
      if (aria && title) {
        if (aria.endsWith(title)) {
          artists = aria.slice(0, aria.length - title.length).trim();
        } else {
          const idx = aria.indexOf(title);
          if (idx > 0) artists = aria.slice(0, idx).trim();
        }
      }
      if (artists) source = 'fallback';
    }

    if (!id) {
      const aria = row.getAttribute('aria-label') || '';
      const img = row.querySelector('img[alt^="Трек "]');
      const alt = img?.getAttribute('alt') || '';
      if (aria || alt || title) {
        id = `fallback::${aria}::${alt}::${title}`;
        source = 'fallback';
      }
    }

    if (!title) return null;

    return {
      id,
      title,
      artists,
      source
    };
  };

  const extractTracks = () => {
    const rows = document.querySelectorAll('.CommonTrack_root__i6shE');

    rows.forEach(row => {
      const parsed = parseRow(row);
      if (!parsed || !parsed.id) return;
      if (tracks.has(parsed.id)) return;
      tracks.set(parsed.id, parsed);
    });
  };

  const getStats = () => {
    const arr = Array.from(tracks.values());
    const fallbackCount = arr.filter(x => x.source === 'fallback').length;
    const normalCount = arr.filter(x => x.source === 'normal').length;
    return { total: arr.length, normalCount, fallbackCount };
  };

  const formatTracks = () => {
    const arr = Array.from(tracks.values());

    const txt = arr.map(t => `${t.artists || ''} - ${t.title}`).join('\n');

    const esc = s => `"${String(s || '').replace(/"/g, '""')}"`;
    const csv = [
      'Artist,Title,Source',
      ...arr.map(t => `${esc(t.artists)},${esc(t.title)},${esc(t.source)}`)
    ].join('\n');

    return { txt, csv, arr };
  };

  const downloadFile = (content, filename, type) => {
    const blob = new Blob([content], { type });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = filename;
    a.click();
    setTimeout(() => URL.revokeObjectURL(url), 1000);
  };

  const renderPanel = (state = '') => {
    const panel = getPanel();
    const { total, normalCount, fallbackCount } = getStats();

    panel.innerHTML = `
      <div class="h">
        <div class="l">Track Collector+</div>
        <div class="dot ${running ? 'a' : ''}"></div>
      </div>

      <div class="c">${total}</div>

      <div class="s">
        ${state || (running ? 'Scroll manually — collecting tracks' : 'Finished')}
      </div>

      <div class="m">
        normal: ${normalCount} · fallback: ${fallbackCount}
      </div>

      ${
        running
          ? `
            <button id="ym-settle" class="p">Finalize capture</button>
            <button id="ym-stop">Stop</button>
          `
          : `
            <button id="ym-dl-txt" class="p">Download .txt</button>
            <button id="ym-dl-csv">Download .csv</button>
          `
      }

      <div class="f">tool by <span>idaniil24 + patched</span></div>
    `;

    if (running) {
      const stopBtn = document.getElementById('ym-stop');
      const settleBtn = document.getElementById('ym-settle');

      if (stopBtn) {
        stopBtn.onclick = () => {
          running = false;
          renderPanel('Stopped');
        };
      }

      if (settleBtn) {
        settleBtn.onclick = settleCapture;
      }
    } else {
      const txtBtn = document.getElementById('ym-dl-txt');
      const csvBtn = document.getElementById('ym-dl-csv');

      if (txtBtn) {
        txtBtn.onclick = () => {
          const { txt } = formatTracks();
          downloadFile(txt, 'tracks-full.txt', 'text/plain;charset=utf-8');
        };
      }

      if (csvBtn) {
        csvBtn.onclick = () => {
          const { csv } = formatTracks();
          downloadFile(csv, 'tracks-full.csv', 'text/csv;charset=utf-8');
        };
      }
    }
  };

  const sleep = ms => new Promise(r => setTimeout(r, ms));

  const settleCapture = async () => {
    renderPanel('Finalizing capture...');
    const start = Date.now();
    let last = -1;

    while (Date.now() - start < CFG.settleMs) {
      extractTracks();
      if (tracks.size === last) break;
      last = tracks.size;
      renderPanel('Finalizing capture...');
      await sleep(CFG.settleTickMs);
    }

    running = false;
    renderPanel('Ready to download');
    window.__ym_tracks_full = formatTracks().arr;
    console.log('Collected tracks:', formatTracks().arr.length);
  };

  const scroller =
    document.querySelector('[data-virtuoso-scroller="true"]') ||
    document.scrollingElement ||
    document.documentElement;

  const debounce = (fn, ms) => {
    let t;
    return (...args) => {
      clearTimeout(t);
      t = setTimeout(() => fn(...args), ms);
    };
  };

  const onScroll = debounce(() => {
    if (!running) return;
    extractTracks();
    renderPanel('Collecting tracks...');
  }, CFG.debounceMs);

  const oldPanel = document.getElementById('ym-scraper');
  if (oldPanel) oldPanel.remove();

  injectStyles();
  renderPanel();
  extractTracks();

  if (scroller) {
    scroller.addEventListener('scroll', onScroll, { passive: true });
  }
})();
