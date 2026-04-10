---
layout: page
title: Agenda
permalink: /agenda/
---

<p>
  Gli eventi del gruppo sono pubblicati su
  <a href="https://mobilizon.it/@badmintonmotus" target="_blank">Mobilizon</a>.
  Di seguito gli eventi visibili pubblicamente in programma.
</p>

<div id="eventi-lista"><em>Caricamento eventi in corso…</em></div>

<script>
  const MOBILIZON_URL = "https://mobilizon.it/api";
  const GROUP_NAME    = "badmintonmotus";

  const QUERY = `
    query FetchGroupEvents($name: String!, $afterDateTime: DateTime) {
      group(preferredUsername: $name) {
        organizedEvents(afterDatetime: $afterDateTime) {
          elements {
            title
            beginsOn
            endsOn
            description
            url
            physicalAddress {
              description
              street
              locality
            }
          }
        }
      }
    }
  `;

  function formatDate(iso) {
    if (!iso) return "";
    const d = new Date(iso);
    return d.toLocaleString("it-IT", {
      weekday: "long",
      year:    "numeric",
      month:   "long",
      day:     "numeric",
      hour:    "2-digit",
      minute:  "2-digit"
    });
  }

  function renderAddress(addr) {
    if (!addr) return "";
    const parts = [addr.description, addr.street, addr.locality]
      .filter(Boolean)
      .join(", ");
    return parts ? `<p><strong>Luogo:</strong> ${parts}</p>` : "";
  }

  function stripHtml(html) {
    const tmp = document.createElement("div");
    tmp.innerHTML = html || "";
    return tmp.textContent || tmp.innerText || "";
  }

  async function loadEvents() {
    const container = document.getElementById("eventi-lista");
    try {
      const response = await fetch(MOBILIZON_URL, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          query: QUERY,
          variables: {
            name: GROUP_NAME,
            afterDateTime: new Date().toISOString()
          }
        })
      });

      const json     = await response.json();
      const elements = json?.data?.group?.organizedEvents?.elements ?? [];

      if (elements.length === 0) {
        container.innerHTML = "<p>Nessun evento in programma al momento.</p>";
        return;
      }

      container.innerHTML = elements.map(ev => `
        <div style="margin-bottom: 2rem; border-bottom: 1px solid #ccc; padding-bottom: 1rem;">
          <h2><a href="${ev.url}" target="_blank">${ev.title}</a></h2>
          <p><strong>Inizio:</strong> ${formatDate(ev.beginsOn)}</p>
          ${ev.endsOn ? `<p><strong>Fine:</strong> ${formatDate(ev.endsOn)}</p>` : ""}
          ${renderAddress(ev.physicalAddress)}
          ${ev.description ? `<p>${stripHtml(ev.description)}</p>` : ""}
        </div>
      `).join("");

    } catch (err) {
      container.innerHTML = "<p>Impossibile caricare gli eventi. Riprova più tardi.</p>";
      console.error(err);
    }
  }

  loadEvents();
</script>