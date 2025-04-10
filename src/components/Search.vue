<template>
  <div class="search">
    <h1>Поиск имен</h1>
    <div class="ssearch__container">
      <div class="input-wrapper">
        <input
          type="text"
          v-model="searchQuery"
          @input="handleInput"
          @keydown="handleKeyNavigation"
          placeholder="Введите имя..."
          class="ssearch__input"
          ref="inputRef"
        />
        <button 
          v-if="searchQuery.trim()" 
          @click="clearSearch" 
          class="clear-button"
          title="Очистить поиск"
        >
          <span class="clear-icon">×</span>
        </button>
      </div>
      <transition name="fade">
        <div v-if="showSuggestions" class="suggestions">
          <div v-if="filteredNames.length === 0" class="no-results">
            Ничего не найдено
          </div>
          <ul v-else class="suggestions__list">
            <li
              v-for="(name, index) in filteredNames"
              :key="index"
              @click="selectName(name)"
              :class="{ 'selected': index === selectedIndex }"
              v-memo="[name, index === selectedIndex]"
            >
              {{ name }}
            </li>
          </ul>
        </div>
      </transition>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, onUnmounted, nextTick, shallowRef } from 'vue';

const namesList = [
  "Alice", "Bob", "Alex", "Алина", "Максим",
  "Jennifer", "David", "Michael", "Наталья", "Сергей",
  "Anna", "Pavel", "Екатерина", "John", "Maria",
  "Victor", "Олег", "Татьяна", "Christopher", "Елена",
  "Daniel", "Александр", "Ирина", "Sarah", "Андрей",
  "Kate", "Emily", "Elizabeth", "Светлана", "Игорь",
  "Anton", "Ольга", "Наталья", "Alexey", "Евгения",
  "Maxim", "Екатерина", "Ivan", "Ольга", "Damir", "Дмитрий"
];

// Используем shallowRef для оптимизации, так как namesList статический
const namesListRef = shallowRef(namesList);

// карта транслитерации (используем объект для O(1) доступа)
const translitMap = {
  'a': 'а', 'b': 'б', 'c': 'ц', 'd': 'д', 'e': 'е', 'f': 'ф',
  'g': 'г', 'h': 'х', 'i': 'и', 'j': 'й', 'k': 'к', 'l': 'л',
  'm': 'м', 'n': 'н', 'o': 'о', 'p': 'п', 'q': 'к', 'r': 'р',
  's': 'с', 't': 'т', 'u': 'у', 'v': 'в', 'w': 'в', 'x': 'кс',
  'y': 'й', 'z': 'з',
  'а': 'a', 'б': 'b', 'в': 'v', 'г': 'g', 'д': 'd', 'е': 'e',
  'ё': 'yo', 'ж': 'zh', 'з': 'z', 'и': 'i', 'й': 'y', 'к': 'k',
  'л': 'l', 'м': 'm', 'н': 'n', 'о': 'o', 'п': 'p', 'р': 'r',
  'с': 's', 'т': 't', 'у': 'u', 'ф': 'f', 'х': 'h', 'ц': 'ts',
  'ч': 'ch', 'ш': 'sh', 'щ': 'sch', 'ъ': '', 'ы': 'y', 'ь': '',
  'э': 'e', 'ю': 'yu', 'я': 'ya'
};

const translitCache = new Map();

const searchQuery = ref('');
const showSuggestions = ref(false);
const selectedIndex = ref(-1);
const inputRef = ref(null);
const debounceTimeout = ref(null);

const getQuery = (query) => {
  if (translitCache.has(query)) {
    return translitCache.get(query);
  }

  const result = query.toLowerCase().split('').map(char => {
    return translitMap[char] || char;
  }).join('');

  translitCache.set(query, result);

  return result;
};

const filteredNames = computed(() => {
  const query = searchQuery.value.trim().toLowerCase();

  if (!query) return [];

  const currentQuery = getQuery(query);

  return namesListRef.value.filter(name => {
    const lowerName = name.toLowerCase();
    return lowerName.startsWith(query) || lowerName.startsWith(currentQuery);
  });
});

const handleInput = () => {
  if (debounceTimeout.value) {
    clearTimeout(debounceTimeout.value);
  }

  debounceTimeout.value = setTimeout(() => {
    selectedIndex.value = -1;
    showSuggestions.value = searchQuery.value.trim() !== '';
  }, 300);
};

const clearSearch = () => {
  searchQuery.value = '';
  showSuggestions.value = false;
  selectedIndex.value = -1;
  nextTick(() => {
    inputRef.value.focus();
  });
};

const selectName = (name) => {
  searchQuery.value = name;
  showSuggestions.value = false;
  selectedIndex.value = -1;

  nextTick(() => {
    inputRef.value.focus();
  });
};

const handleKeyNavigation = (e) => {
  if (!showSuggestions.value) return;

  switch (e.key) {
    case 'ArrowDown':
      e.preventDefault();
      selectedIndex.value = (selectedIndex.value < filteredNames.value.length - 1) 
        ? selectedIndex.value + 1 
        : filteredNames.value.length - 1;
      break;
    
    case 'ArrowUp':
      e.preventDefault();
      selectedIndex.value = (selectedIndex.value > 0) 
        ? selectedIndex.value - 1 
        : 0;
      break;
    
    case 'Enter':
      if (selectedIndex.value >= 0 && selectedIndex.value < filteredNames.value.length) {
        selectName(filteredNames.value[selectedIndex.value]);
      }
      break;
    
    case 'Escape':
      showSuggestions.value = false;
      selectedIndex.value = -1;
      break;
  }
};

const handleClickOutside = (e) => {
  if (inputRef.value && !inputRef.value.contains(e.target)) {
    showSuggestions.value = false;
  }
};

watch(filteredNames, () => {
  selectedIndex.value = -1;
}, { flush: 'post' });

onMounted(() => {
  document.addEventListener('click', handleClickOutside);
});

onUnmounted(() => {
  document.removeEventListener('click', handleClickOutside);

  if (debounceTimeout.value) {
    clearTimeout(debounceTimeout.value);
  }
});
</script>

<style scoped>
.search {
  max-width: 500px;
  margin: 0 auto;
  font-family: Arial, sans-serif;
}

h1 {
  text-align: center;
  margin-bottom: 20px;
  color: #333;
}

.ssearch__container {
  position: relative;
}

.input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.ssearch__input {
  width: 100%;
  padding: 12px 15px;
  padding-right: 40px; 
  font-size: 16px;
  border: 2px solid #ddd;
  border-radius: 6px;
  box-sizing: border-box;
  outline: none;
  transition: border-color 0.3s;
  -webkit-appearance: none;
}

.ssearch__input:focus {
  border-color: #4a90e2;
}

.clear-button {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  background: none;
  border: none;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background-color: #e0e0e0;
  transition: background-color 0.2s;
}

.clear-button:hover {
  background-color: #ccc;
}

.clear-icon {
  font-size: 18px;
  line-height: 1;
  color: #666;
}

.suggestions {
  position: absolute;
  width: 100%;
  max-height: 300px;
  overflow-y: auto;
  margin-top: 5px;
  background: white;
  border: 1px solid #ddd;
  border-radius: 6px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  z-index: 1000;
  will-change: opacity, transform;
}

.suggestions__list {
  list-style: none;
  margin: 0;
  padding: 0;
}

.suggestions__list li {
  padding: 10px 15px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.suggestions__list li:hover,
.suggestions__list li.selected {
  background-color: #f0f7ff;
}

.no-results {
  padding: 15px;
  text-align: center;
  color: #888;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.25s ease, transform 0.25s ease;
  will-change: opacity, transform;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

@media (max-width: 768px) {
  .search {
    padding: 0 15px;
  }
  
  .ssearch__input {
    padding: 15px;
    padding-right: 45px;
    font-size: 18px; 
    -webkit-tap-highlight-color: transparent;
  }
  
  .clear-button {
    width: 28px;
    height: 28px;
    right: 12px;
  }
  
  .clear-icon {
    font-size: 20px;
  }
  
  .suggestions__list li {
    padding: 15px;
    font-size: 16px;
  }
}
</style>