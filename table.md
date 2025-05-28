---
layout: default
title: Creature Table
---

<h2>Creature Table (Spoiler Controlled)</h2>

<label for="chapterInput">Your current chapter:</label>
<input type="number" id="chapterInput" placeholder="e.g. 12" value="1"/>

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
              <!-- Removed First Mention field -->
              <!-- Excerpt hover popup -->
              <span class="excerpt-hover" data-chapter="{{ entry.excerpt.chapter }}">
                <strong style="cursor: pointer; text-decoration: underline dotted;">Excerpt</strong>
                <span class="excerpt-popup">
                  "{{ entry.excerpt.text }}"
                </span>
              </span>
              <!-- Notables toggle section -->
              <div class="notables-section">
                <p>
                  <strong class="notables-toggle" style="cursor: pointer; text-decoration: underline dashed;">Notables</strong>
                </p>
                <ul class="notables-list" style="display: none;">
                  {% for creature in entry.notable_creatures %}
                    <li data-chapter="{{ creature.chapter }}">
                      <span class="notable-hover" style="cursor: pointer; text-decoration: underline dotted;">
                        {{ creature.name }}
                        <span class="notable-popup">
                          "{{ creature.description }}"
                        </span>
                      </span>
                    </li>
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

<style>
/* Excerpt hover popup styles */
.excerpt-hover {
  position: relative;
  display: inline-block;
}
.excerpt-popup {
  display: none;
  position: absolute;
  left: 0;
  top: 1.5em;
  z-index: 10;
  background: #222;
  color: #fff;
  padding: 0.7em 1em;
  border-radius: 6px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.2);
  min-width: 200px;
  max-width: 350px;
  font-size: 0.95em;
  white-space: normal;
}
.excerpt-hover:hover .excerpt-popup {
  display: block;
}

/* Notable hover popup styles */
.notable-hover {
  position: relative;
  display: inline-block;
}
.notable-popup {
  display: none;
  position: absolute;
  left: 0;
  top: 1.5em;
  z-index: 10;
  background: #222;
  color: #fff;
  padding: 0.7em 1em;
  border-radius: 6px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.2);
  min-width: 200px;
  max-width: 350px;
  font-size: 0.95em;
  white-space: normal;
}
.notable-hover:hover .notable-popup {
  display: block;
}

.notables-list {
  padding-left: 1.2em; /* Ensures bullets align correctly */
  list-style-position: outside; /* Default, but explicit is better */
}

.notables-list li {
  display: list-item;
  vertical-align: top;
}

.notables-list li .notable-hover {
  display: inline-block;
  vertical-align: top;
}

</style>

<script>
function updateVisibility(userChapter) {
  document.querySelectorAll("td").forEach(td => {
    const contentDiv = td.querySelector(".creature-content");
    const naPlaceholder = td.querySelector(".na-placeholder");
    
    if (contentDiv) {
      let hasVisibleContent = false;
      
      // Check all content elements except notables
      const elementsToCheck = [
        ...contentDiv.querySelectorAll("p[data-chapter]"),
        ...contentDiv.querySelectorAll(".excerpt-hover[data-chapter]")
      ];
      
      elementsToCheck.forEach(el => {
        if (!el) return;
        const chapter = parseInt(el.dataset.chapter, 10);
        const shouldShow = chapter <= userChapter;
        el.style.display = shouldShow ? "" : "none";
        if (shouldShow) hasVisibleContent = true;
      });

      // Handle notables section
      const notablesSection = contentDiv.querySelector(".notables-section");
      if (notablesSection) {
        const notablesList = notablesSection.querySelector(".notables-list");
        const notablesToggle = notablesSection.querySelector(".notables-toggle");
        let anyVisibleNotable = false;
        if (notablesList) {
          // Show/hide each notable creature individually
          Array.from(notablesList.querySelectorAll("li[data-chapter]")).forEach(li => {
            const chapter = parseInt(li.dataset.chapter, 10);
            // Hide if no name or no chapter
            if (!li.textContent.trim() || isNaN(chapter)) {
              li.style.display = "none";
              return;
            }
            if (chapter <= userChapter) {
              li.style.display = "";
              anyVisibleNotable = true;
            } else {
              li.style.display = "none";
            }
          });
          // Hide notables list if toggled closed (default), but only show toggle if any visible
          if (!anyVisibleNotable) {
            notablesSection.style.display = "none";
          } else {
            notablesSection.style.display = "";
            hasVisibleContent = true;
          }
        } else {
          notablesSection.style.display = "none";
        }
      }

      // Toggle N/A placeholder
      if (naPlaceholder) {
        naPlaceholder.style.display = hasVisibleContent ? "none" : "";
      }
    }
  });
}

// Add click toggling for notables
document.addEventListener("DOMContentLoaded", function() {
  const saved = parseInt(localStorage.getItem("userChapter"), 10);
  if (!isNaN(saved)) {
    document.getElementById("chapterInput").value = saved;
    updateVisibility(saved);
  }
  // Notables toggle logic
  document.querySelectorAll(".notables-toggle").forEach(function(toggle) {
    toggle.addEventListener("click", function() {
      const ul = this.closest(".notables-section").querySelector(".notables-list");
      if (ul) {
        ul.style.display = (ul.style.display === "none" || ul.style.display === "") ? "block" : "none";
      }
    });
  });

  // Notables popup logic
  document.querySelectorAll(".notable-hover").forEach(function(span) {
    span.addEventListener("click", function(event) {
      event.stopPropagation();
      // Hide any other open popups
      document.querySelectorAll(".notable-popup").forEach(function(popup) {
        if (popup !== span.querySelector(".notable-popup")) {
          popup.style.display = "none";
        }
      });
      const popup = span.querySelector(".notable-popup");
      if (popup) {
        popup.style.display = (popup.style.display === "block") ? "none" : "block";
      }
    });
  });

  // Hide popup when clicking outside
  document.addEventListener("click", function() {
    document.querySelectorAll(".notable-popup").forEach(function(popup) {
      popup.style.display = "none";
    });
  });
});

document.getElementById("chapterInput").addEventListener("input", function() {
  const chapter = parseInt(this.value, 10);
  if (!isNaN(chapter)) {
    localStorage.setItem("userChapter", chapter);
    updateVisibility(chapter);
  }
});
</script>