---
layout: default
title: Creature Lookup
---

<h2>Creature Lookup</h2>

<div class="lookup-container">
  <div class="select-group">
    <label for="rankSelect">Select Rank:</label>
    <select id="rankSelect">
      <option value="">-- Choose Rank --</option>
      {% for rank in site.data.rank %}<option value="{{ rank.rank | escape }}">{{ rank.rank | escape }}</option>{% endfor %}
    </select>
  </div>

  <div class="select-group">
    <label for="classSelect">Select Class:</label>
    <select id="classSelect">
      <option value="">-- Choose Class --</option>
      {% for class in site.data.class %}<option value="{{ class.class | escape }}">{{ class.class | escape }}</option>{% endfor %}
    </select>
  </div>
</div>

<div id="infoDisplay" style="margin-top:2em;"></div>

<style>
.lookup-container {
  display: flex;
  gap: 2em;
  margin: 1em 0;
}

.select-group {
  display: flex;
  align-items: center;
  gap: 0.5em;
}

select {
  padding: 0.3em;
  border-radius: 4px;
}
</style>

<script>
const rankData = {
  {% for rank in site.data.rank %}
    "{{ rank.rank | escape }}": {{ rank | jsonify }},
  {% endfor %}
};
const classData = {
  {% for class in site.data.class %}
    "{{ class.class | escape }}": {{ class | jsonify }},
  {% endfor %}
};
const creaturesData = [
  {% for creature in site.data.creatures %}
    {{ creature | jsonify }},
  {% endfor %}
];

function displayInfo() {
  const rank = document.getElementById('rankSelect').value;
  const cls = document.getElementById('classSelect').value;
  let html = '';

  if (rank && rankData[rank]) {
    html += `<h3>Rank: ${rank}</h3>`;
    if (rankData[rank].description)
      html += `<p>${rankData[rank].description}</p>`;
  }
  if (cls && classData[cls]) {
    html += `<h3>Class: ${cls}</h3>`;
    if (classData[cls].description)
      html += `<p>${classData[cls].description}</p>`;
  }

  if (rank && cls) {
    const creature = creaturesData.find(c => c.rank === rank && c.class === cls);
    if (creature) {
      html += `<h4>Notable Creatures:</h4>`;
      if (creature.notable_creatures && creature.notable_creatures.length > 0) {
        html += '<ul>';
        creature.notable_creatures.forEach(nc => {
          if (nc.name && nc.name.trim()) {
            html += `<li><strong>${nc.name}</strong>`;
            if (nc.description && nc.description.trim()) {
              html += `: ${nc.description}`;
            }
            if (nc.chapter && nc.chapter.toString().trim()) {
              html += ` <em>(Chapter ${nc.chapter})</em>`;
            }
            html += `</li>`;
          }
        });
        html += '</ul>';
      } else {
        html += '<p>No notable creatures listed.</p>';
      }
    } else {
      html += '<p>No data for this rank/class combination.</p>';
    }
  }

  document.getElementById('infoDisplay').innerHTML = html;
}

document.getElementById('rankSelect').addEventListener('change', displayInfo);
document.getElementById('classSelect').addEventListener('change', displayInfo);
</script>