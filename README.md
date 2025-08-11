(() => {
  const SERVICES = [
    {
      id: 'basic-build',
      name: 'Basic Build',
      price: 100,
      details: [
        'Assemble from your parts',
        'Cable management',
        'BIOS setup & Windows install',
        'Driver updates'
      ]
    },
    {
      id: 'intermediate-build',
      name: 'Intermediate Build',
      price: 130,
      details: [
        'Everything in Basic',
        'RGB/fan controller setup',
        'Wi‑Fi/Bluetooth or PCIe add‑ons',
        'Dual storage config, light OC (if supported)'
      ]
    },
    {
      id: 'advanced-build',
      name: 'Advanced Build',
      price: 160,
      details: [
        'Everything in Intermediate',
        'AIO liquid cooler install',
        'High‑end GPU/PSU setup',
        'Advanced BIOS & stress testing'
      ]
    },
    {
      id: 'consultation',
      name: 'PC Build Consultation (Phone/Video)',
      price: 35,
      details: [
        '45‑minute call',
        'Custom parts list for your budget',
        'Fee credited if you book a build'
      ]
    },
    {
      id: 'repair-upgrade',
      name: 'Repair / Upgrade (Desktop only)',
      price: 0,
      details: [
        '$40 pre‑diagnostic (credited if you proceed)',
        'Hardware swaps, OS/driver reinstalls, cleanups',
        'Typical turnaround: 1–3 business days'
      ]
    }
  ];

  const $ = s => document.querySelector(s);
  const $$ = s => document.querySelectorAll(s);

  function renderServices() {
    const grid = $("#services-grid");
    if (!grid) return;
    grid.innerHTML = SERVICES.map(s => `
      <div class="card">
        <h3>${s.name}</h3>
        <div class="price">${s.price ? `$${s.price} flat` : 'Varies'}</div>
        <ul>${s.details.map(d => `<li><small>${d}</small></li>`).join('')}</ul>
        <div class="section">
          <button class="btn" type="button" onclick="gotoBook('${s.id}')">Book ${s.price ? '' : 'Repair'}</button>
        </div>
      </div>
    `).join('');
    // Fill select options
    const sel = $("#service");
    if (sel) {
      sel.innerHTML = SERVICES.map(s => `<option value="${s.id}">${s.name}</option>`).join('');
    }
  }

  function goto(view) {
    $$(".view").forEach(v => v.hidden = true);
    const showView = $(`#view-${view}`);
    if (showView) showView.hidden = false;
    $$("nav button").forEach(b => {
      const active = b.dataset.view === view;
      b.classList.toggle('active', active);
      b.setAttribute('aria-pressed', active ? 'true' : 'false');
    });
    window.scrollTo({ top: 0, behavior: 'smooth' });
  }

  // Expose globally for button onclick
  window.gotoBook = function (serviceId) {
    goto('book');
    const sel = $("#service");
    if (sel) sel.value = serviceId;
    updateEstimate();
  };

  function servicePrice(id) {
    const s = SERVICES.find(x => x.id === id);
    if (!s) return 0;
    // Repairs have no flat price; estimate starts at diagnostic
    return s.id === 'repair-upgrade' ? 40 : s.price;
  }

  function updateEstimate() {
    const sel = $("#service");
    const home = $("#homeVisit");
    if (!sel || !home) return;
    const selectedService = sel.value;
    const homeVisit = home.value === 'yes';
    let total = servicePrice(selectedService);
    if (homeVisit) total += 25;
    const estimateElem = $("#estimate");
    if (estimateElem) estimateElem.textContent = `$${total}`;
  }

  function saveRequest(data) {
    const key = 'lewsco_requests';
    const current = JSON.parse(localStorage.getItem(key) || '[]');
    current.unshift(data);
    localStorage.setItem(key, JSON.stringify(current));
  }

  function renderRequests() {
    const key = 'lewsco_requests';
    const list = JSON.parse(localStorage.getItem(key) || '[]');
    const wrap = $("#requests");
    if (!wrap) return;
    if (!list.length) {
      wrap.innerHTML = '<p class="muted">No saved requests yet.</p>';
      return;
    }
    wrap.innerHTML = list.map((r, i) => `
      <div class="card">
        <div>
          <strong>${r.name}</strong> • <small>${r.email} • ${r.phone}</small><br>
          <small>${new Date(r.createdAt).toLocaleString()}</small><br>
          <small>Service: ${r.serviceName}${r.homeVisit === 'yes' ? ' • Home visit' : ''}</small><br>
          <small>Date: ${r.date} ${r.time || ''}</small><br>
          <small>Address: ${r.address || '—'}</small><br>
          <small>Notes: ${r.notes || '—'}</small><br>
          <strong>Estimate: $${r.estimate}</strong>
        </div>
        <div class="actions">
          <button class="btn alt" type="button" onclick="emailQuote(${i})">Email</button>
          <button class="btn alt" type="button" onclick="copyQuote(${i})">Copy</button>
          <button class="btn" type="button" onclick="deleteReq(${i})">Delete</button>
        </div>
      </div>
    `).join('');
  }

  // Expose globally for button onclick
  window.deleteReq = function (i) {
    const key = 'lewsco_requests';
    const list = JSON.parse(localStorage.getItem(key) || '[]');
    list.splice(i, 1);
    localStorage.setItem(key, JSON.stringify(list));
    renderRequests();
  };

  // Expose globally for button onclick
  window.emailQuote = function (i) {
    const key = 'lewsco_requests';
    const list = JSON.parse(localStorage.getItem(key) || '[]');
    const r = list[i];
    const subject = encodeURIComponent(`LewSCO.Tech Estimate — ${r.serviceName}`);
    const body = encodeURIComponent(`Hi ${r.name},

Thanks for your request with LewSCO.Tech. Here is your estimate:

Service: ${r.serviceName}
Home visit: ${r.homeVisit === 'yes' ? 'Yes (+$25)' : 'No'}
Preferred: ${r.date} ${r.time || ''}
Estimate: $${r.estimate}

Notes:
${r.notes || '-'}

We’ll confirm by text/email shortly. — LewSCO.Tech`);
    window.location.href = `mailto:${r.email}?subject=${subject}&body=${body}`;
  };

  // Expose globally for button onclick
  window.copyQuote = function (i) {
    const key = 'lewsco_requests';
    const list = JSON.parse(localStorage.getItem(key) || '[]');
    const r = list[i];
    const text = `LewSCO.Tech Estimate — ${r.serviceName}
Name: ${r.name}
Phone: ${r.phone}
Email: ${r.email}
Home visit: ${r.homeVisit}
Preferred: ${r.date} ${r.time || ''}
Address: ${r.address || '-'}
Notes: ${r.notes || '-'}
Estimate: $${r.estimate}`;
    navigator.clipboard.writeText(text)
      .then(() => alert('Copied to clipboard'))
      .catch(() => alert('Copy failed'));
  };

  function currentServiceName() {
    const sel = $("#service");
    if (!sel) return '';
    const id = sel.value;
    const s = SERVICES.find(x => x.id === id);
    return s ? s.name : id;
  }

  document.addEventListener('DOMContentLoaded', () => {
    renderServices();
    goto('home');
    const yearElem = $("#year");
    if (yearElem) yearElem.textContent = new Date().getFullYear();

    const serviceSel = $("#service");
    if (serviceSel) serviceSel.addEventListener('change', updateEstimate);

    const homeVisitSel = $("#homeVisit");
    if (homeVisitSel) homeVisitSel.addEventListener('change', updateEstimate);

    updateEstimate();

    const bookingForm = $("#booking-form");
    if (bookingForm) {
      bookingForm.addEventListener('submit', (e) => {
        e.preventDefault();
        const data = Object.fromEntries(new FormData(e.target).entries());
        data.serviceName = currentServiceName();
        const estElem = document.querySelector('#estimate');
        let estimate = 0;
        if (estElem && estElem.textContent) {
          estimate = Number(estElem.textContent.replace('$', '')) || 0;
        }
        data.estimate = estimate;
        data.createdAt = new Date().toISOString();
        saveRequest(data);
        alert('Request saved locally. We will contact you to confirm.');
        e.target.reset();
        renderRequests();
        updateEstimate();
      });
    }

    const emailBtn = $("#email-btn");
    if (emailBtn) {
      emailBtn.addEventListener('click', () => {
        const form = document.querySelector('#booking-form');
        if (!form) return;
        const data = Object.fromEntries(new FormData(form).entries());
        data.serviceName = currentServiceName();
        const estElem = document.querySelector('#estimate');
        let estimate = 0;
        if (estElem && estElem.textContent) {
          estimate = Number(estElem.textContent.replace('$', '')) || 0;
        }
        const subject = encodeURIComponent(`LewSCO.Tech Estimate — ${data.serviceName}`);
        const body = encodeURIComponent(`Hi ${data.name || ''},

Here is your estimate (not final):

Service: ${data.serviceName}
Home visit: ${data.homeVisit === 'yes' ? 'Yes (+$25)' : 'No'}
Preferred: ${data.date || ''} ${data.time || ''}
Estimate: $${estimate}

Notes:
${data.notes || '-'}

— LewSCO.Tech`);
        const to = encodeURIComponent(data.email || 'LewSCO.Tech@gmail.com');
        window.location.href = `mailto:${to}?subject=${subject}&body=${body}`;
      });
    }

    // Nav buttons
    document.querySelectorAll('nav button').forEach(btn => {
      btn.addEventListener('click', () => goto(btn.dataset.view));
    });

    // Initial render of requests if needed
    renderRequests();
  });
})();
