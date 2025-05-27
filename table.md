---
layout: default
title: Creature Table
---

<h2>Creature Table (Spoiler Controlled)</h2>

<label for="chapterInput">Your current chapter:</label>
<input type="number" id="chapterInput" placeholder="e.g. 12" />

<table id="creatureTable">
  <thead>
    <tr>
      <th>Rank / Class</th>
      {% assign classes = "Beast,Monster,Demon,Devil,Tyrant,Terror,Titan" | split: "," %}
      {% for class in classes %}
        <th>{{ class }}</th>
      {% endfor %}
    </tr>
  </thead>
  <tbody>
    {% assign ranks = "Dormant,Awakened,Fallen,Corrupted,Great,Cursed,Unholy" | split: "," %}
    {% for rank in ranks %}
    <tr>
      <td><strong>{{ rank }}</strong></td>
      {% for class in classes %}
        {% assign entry = site.data.creatures | where: "rank", rank | where: "class", class | first %}
        <td>
          {% if entry %}
            <div class="creature-content">
              <p data-chapter="{{ entry.description.min_chapter }}">
                <em>{{ entry.description.text }}</em>
              </p>
              <p data-chapter="{{ entry.first_mention.chapter }}">
                <strong>First Mention:</strong> {{ entry.first_mention.text }}
              </p>
              <p data-chapter="{{ entry.excerpt.chapter }}">
                <strong>Excerpt:</strong> "{{ entry.excerpt.text }}"
              </p>
              <div class="notables-section" data-chapter="{{ entry.notable_creatures | map: 'chapter' | min }}">
                <p>
                  <strong>Notables:</strong>
                </p>
                <ul>
                  {% for creature in entry.notable_creatures %}
                  <li data-chapter="{{ creature.chapter }}">{{ creature.name }}</li>
                  {% endfor %}
                </ul>
              </div>
            </div>
            <p class="na-placeholder" style="display: none; opacity: 0.4;">N/A</p>
          {% else %}
            <p style="opacity: 0.4;">N/A</p>
          {% endif %}
        </td>
      {% endfor %}
    </tr>
    {% endfor %}
  </tbody>
</table>

<script>
function updateVisibility(userChapter) {
  document.querySelectorAll("td").forEach(td => {
    const contentDiv = td.querySelector(".creature-content");
    const naPlaceholder = td.querySelector(".na-placeholder");
    
    if (contentDiv) {
      let hasVisibleContent = false;
      
      // Check all content elements
      const elementsToCheck = [
        ...contentDiv.querySelectorAll("p[data-chapter]"),
        contentDiv.querySelector(".notables-section")
      ];
      
      elementsToCheck.forEach(el => {
        if (!el) return;
        
        const chapter = parseInt(el.dataset.chapter, 10);
        const shouldShow = chapter <= userChapter;
        
        // For notable sections, we need additional checks
        if (el.classList.contains('notables-section')) {
          const anyVisibleNotables = Array.from(el.querySelectorAll("li[data-chapter]"))
            .some(li => parseInt(li.dataset.chapter, 10) <= userChapter);
          el.style.display = anyVisibleNotables ? "" : "none";
          if (anyVisibleNotables) hasVisibleContent = true;
        } else {
          el.style.display = shouldShow ? "" : "none";
          if (shouldShow) hasVisibleContent = true;
        }
      });
      
      // Toggle N/A placeholder
      if (naPlaceholder) {
        naPlaceholder.style.display = hasVisibleContent ? "none" : "";
      }
    }
  });
}

document.getElementById("chapterInput").addEventListener("input", function() {
  const chapter = parseInt(this.value, 10);
  if (!isNaN(chapter)) {
    localStorage.setItem("userChapter", chapter);
    updateVisibility(chapter);
  }
});

document.addEventListener("DOMContentLoaded", function() {
  const saved = parseInt(localStorage.getItem("userChapter"), 10);
  if (!isNaN(saved)) {
    document.getElementById("chapterInput").value = saved;
    updateVisibility(saved);
  }
});
</script>