<template>
    <v-container fluid class="pa-4">
        <!-- Верхняя панель -->
        <v-toolbar flat density="comfortable" class="mb-4 rounded-lg bg-grey-lighten-4">
            <v-toolbar-title class="text-subtitle-1 font-weight-bold">
                Разделение счёта
            </v-toolbar-title>
            <v-spacer></v-spacer>
            <v-chip class="ma-1" variant="elevated">
                Итог: {{ calculateOverallTotal().toFixed(2) }} смн.
            </v-chip>
            <v-btn color="primary" class="ml-2" size="small" @click="generateImage">
                Сохранить отчёт
            </v-btn>
        </v-toolbar>

        <v-row>
            <!-- Левая колонка: формы и люди -->
            <v-col cols="12" md="5" class="d-flex flex-column ga-4">
                <!-- Добавление блюда -->
                <v-card elevation="1" class="rounded-lg">
                    <v-card-title class="text-subtitle-2">
                        Добавить блюдо
                    </v-card-title>
                    <v-divider></v-divider>
                    <v-card-text>
                        <v-form @submit.prevent="addItem">
                            <v-row dense class="align-center">
                                <v-col cols="12" sm="6">
                                    <v-text-field
                                        v-model="newItem.name"
                                        label="Блюдо"
                                        required
                                        density="compact"
                                        hide-details
                                        prepend-inner-icon="mdi-silverware-fork-knife"
                                    />
                                </v-col>
                                <v-col cols="9" sm="4">
                                    <v-text-field
                                        v-model.number="newItem.price"
                                        label="Цена"
                                        type="number"
                                        required
                                        density="compact"
                                        hide-details
                                        prepend-inner-icon="mdi-cash"
                                    />
                                </v-col>
                                <v-col cols="3" sm="2" class="text-right">
                                    <v-btn color="primary" type="submit" size="small">Добавить</v-btn>
                                </v-col>
                            </v-row>
                        </v-form>
                    </v-card-text>
                </v-card>

                <!-- Общие расходы -->
                <v-card elevation="1" class="rounded-lg">
                    <v-card-title class="text-subtitle-2">
                        Общие расходы
                        <v-tooltip text="Эти суммы будут поровну распределены между выбранными людьми.">
                            <template #activator="{ props }">
                                <v-icon v-bind="props" size="18" class="ml-1">mdi-information-outline</v-icon>
                            </template>
                        </v-tooltip>
                    </v-card-title>
                    <v-divider></v-divider>
                    <v-card-text>
                        <v-form @submit.prevent="updateCosts">
                            <v-row dense class="align-center">
                                <v-col cols="6" sm="3">
                                    <v-text-field
                                        v-model.number="costs.bread"
                                        label="Хлеб"
                                        type="number"
                                        min="0"
                                        step="0.01"
                                        density="compact"
                                        hide-details
                                        class="bg-light-green-accent-1 rounded-lg"
                                        prepend-inner-icon="mdi-baguette"
                                    />
                                </v-col>
                                <v-col cols="6" sm="3">
                                    <v-text-field
                                        v-model.number="costs.tea"
                                        label="Чай"
                                        type="number"
                                        min="0"
                                        step="0.01"
                                        density="compact"
                                        hide-details
                                        class="bg-light-green-accent-1 rounded-lg"
                                        prepend-inner-icon="mdi-tea"
                                    />
                                </v-col>
                                <v-col cols="6" sm="3">
                                    <v-text-field
                                        v-model.number="costs.cola"
                                        label="Кола"
                                        type="number"
                                        min="0"
                                        step="0.01"
                                        density="compact"
                                        hide-details
                                        class="bg-light-green-accent-1 rounded-lg"
                                        prepend-inner-icon="mdi-bottle-soda-classic-outline"
                                    />
                                </v-col>
                                <v-col cols="6" sm="3">
                                    <v-text-field
                                        v-model.number="serviceCharge"
                                        label="Обслуживание"
                                        type="number"
                                        min="0"
                                        step="0.01"
                                        density="compact"
                                        hide-details
                                        class="bg-green rounded-lg"
                                        prepend-inner-icon="mdi-receipt-text-outline"
                                    />
                                </v-col>
                                <v-col cols="12" class="text-right">
                                    <v-btn color="primary" type="submit" size="small">Обновить</v-btn>
                                </v-col>
                            </v-row>
                        </v-form>
                    </v-card-text>
                </v-card>

                <!-- Выбор людей -->
                <v-card elevation="1" class="rounded-lg">
                    <v-card-title class="text-subtitle-2">
                        Участники
                    </v-card-title>
                    <v-divider></v-divider>
                    <v-card-text>
                        <v-autocomplete
                            v-model="selectedPeople"
                            :items="people"
                            label="Люди"
                            chips
                            multiple
                            clearable
                            density="compact"
                            hide-details
                        >
                            <template #chip="{ props, item }">
                                <v-chip v-bind="props" class="ma-1" size="small" variant="elevated">
                                    <v-icon start size="16">mdi-account</v-icon>
                                    {{ item.title ?? item }}
                                </v-chip>
                            </template>
                        </v-autocomplete>
                    </v-card-text>
                </v-card>

                <!-- Карточки людей и выбор блюд -->
                <v-row dense>
                    <v-col
                        v-for="person in selectedPeople"
                        :key="person"
                        cols="12"
                        sm="12"
                        md="12"
                    >
                        <v-card class="pa-2 rounded-lg person-card" elevation="1">
                            <v-card-title class="d-flex align-center justify-space-between">
                <span class="text-subtitle-2">
                  <v-icon size="18" class="mr-1">mdi-account</v-icon>{{ person }}
                </span>
                                <v-btn variant="text" size="small" @click="removePerson(person)">Удалить</v-btn>
                            </v-card-title>
                            <v-divider></v-divider>
                            <v-card-text class="pt-2">
                                <v-list density="compact" class="py-0">
                                    <v-list-item
                                        v-for="item in items"
                                        :key="item.name"
                                        class="py-1 px-0"
                                        :ripple="false"
                                    >
                                        <template #prepend>
                                            <v-checkbox
                                                @change="toggleItem(person, item)"
                                                :input-value="isItemSelected(person, item)"
                                                density="compact"
                                                hide-details
                                                class="ma-0"
                                            />
                                        </template>

                                        <v-list-item-title class="d-flex align-center">
                                            <span class="mr-2">{{ item.name }}</span>
                                            <v-chip size="x-small" variant="outlined">
                                                {{ item.price }} смн.
                                            </v-chip>
                                        </v-list-item-title>

                                        <template #append>
                                            <div v-if="isItemSelected(person, item)" class="d-flex align-center ga-2">
                                                <v-text-field
                                                    label="Порции"
                                                    type="number"
                                                    min="0"
                                                    step="0.5"
                                                    density="compact"
                                                    hide-details
                                                    class="portion-field"
                                                    @input="event => setPortion(person, item, event)"
                                                />
                                            </div>
                                        </template>
                                    </v-list-item>
                                </v-list>
                            </v-card-text>
                        </v-card>
                    </v-col>
                </v-row>
            </v-col>

            <!-- Правая колонка: итог и отчёт -->
            <v-col cols="12" md="7" class="d-flex flex-column ga-4">
                <!-- Общая сумма (липкая) -->
                <v-card class="rounded-lg sticky-summary" elevation="2">
                    <v-card-title class="d-flex align-center justify-space-between">
                        <span class="text-subtitle-2">Общая сумма</span>
                        <v-chip size="small" variant="elevated" class="text-button">
                            {{ calculateOverallTotal().toFixed(2) }} смн.
                        </v-chip>
                    </v-card-title>
                </v-card>

                <!-- Отчёт (чек) -->
                <v-card class="rounded-lg" elevation="1">
                    <v-card-title class="report-title">Отчёт</v-card-title>
                    <v-divider></v-divider>
                    <v-card-text ref="reportContent">
                        <!-- Контейнер для отчёта (id НЕ меняется) -->
                        <div id="report" class="report-container elevation-1">
                            <div class="report-header">
                                <div class="report-header-cell">Имя</div>
                                <div class="report-header-cell">Блюда</div>
                                <div class="report-header-cell text-right">Сумма</div>
                            </div>

                            <div class="report-body">
                                <div v-for="person in selectedPeople" :key="person" class="report-row">
                                    <div class="report-cell report-name">
                                        <v-icon size="16" class="mr-1">mdi-account</v-icon>
                                        {{ person }}
                                    </div>
                                    <div class="report-cell">
                                        <ul class="report-dishes">
                                            <li v-for="item in personItems[person]" :key="item.name"
                                                class="report-dish">
                                                <span class="dish-name">{{ item.name }}</span>
                                                <span class="dot"></span>
                                                <span class="dish-price">{{ item.price }} смн.</span>
                                                <span class="times">×</span>
                                                <span class="dish-portion">{{ getPortion(person, item) }} порций</span>
                                            </li>
                                            <li class="report-dish muted">
                                                <v-icon size="14" class="mr-1">mdi-receipt</v-icon>
                                                Обслуживание и прочее: {{ calculateShare() }} смн.
                                            </li>
                                        </ul>
                                    </div>
                                    <div class="report-cell text-right">
                                        {{ calculateTotal(person).toFixed(2) }} смн.
                                    </div>
                                </div>

                                <div class="report-row totals-row">
                                    <div class="report-cell"></div>
                                    <div class="report-cell">
                                        <div class="stacked-costs">
                                            <div><strong>Хлеб:</strong> {{ costs.bread }} смн.</div>
                                            <div><strong>Кола:</strong> {{ costs.cola }} смн.</div>
                                            <div><strong>Чай:</strong> {{ costs.tea }} смн.</div>
                                            <div><strong>Обслуживание:</strong> {{ serviceCharge }} смн.</div>
                                        </div>
                                    </div>
                                    <div class="report-cell text-right">
                                        {{ calculateCosts().toFixed(2) }} смн.
                                    </div>
                                </div>
                            </div>

                            <div class="report-footer">
                                <div class="report-cell footer-label">Итого к оплате:</div>
                                <div class="report-cell footer-value text-right">
                                    {{ calculateOverallTotal().toFixed(2) }} смн.
                                </div>
                            </div>
                        </div>
                    </v-card-text>

                    <v-divider></v-divider>
                    <v-card-actions class="justify-end">
                        <v-btn color="primary" @click="generateImage" size="small">
                            Сохранить как изображение
                        </v-btn>
                    </v-card-actions>
                </v-card>
            </v-col>
        </v-row>
    </v-container>
</template>

<script>
import {reactive} from "vue";
import html2canvas from "html2canvas";

export default {
  data() {
    return {
      newItem: {
        name: '',
        price: 0
      },
      serviceCharge: 0,
      costs: {
        bread: 0,
        tea: 0,
        cola: 0
      },
      people: [
        'Амир Зухуров',
        'Амир Хамидов',
        'Аслиддин Шахобов',
          'Абдурахмон Шерматов',
        'Аюбхон Мамадов',
        'Баходур Турсунов',
        'Бахром  Ерматов',
        'Бахриддин Баховаддинзода',
        'Давлатчон Хакимов',
        'Давронбек Ахмедбеков',
        'Дилмурод Азимов',
        'Диловар Юнусов',
        'Изатилло Ганиев',
        'Искандар Юнусов',
        'Манучехр Музафаров',
        'Мирзоэрач Ибрагимов',
        'Мирзоакмал Султонов',
        'Мубин Юнусов',
        'Муминчон Файзиев',
        'Мухаммадчон Акрамов',
        'Мухаммадчон Мусоев',
        'Мухаммад Вафоев',
        'Мухаммадчон (аналитики нав)',
        'Мухаммадчон Рачабов',
        'Мухриддин Акрамзода',
        'Наимчон Мухамедов',
        'Носиршо Холмамадов',
        'Нуриддин Шахобов',
        'Орифчон Зоидов',
        'Пайрав Юлдошев',
        'Парвиз Каримов',
        'Саидвали Саидисломзода',
        'Салмон Нарзуллоев',
        'Сохибчон',
        'Фирдавс Хочаев',
        'Фирдавсхон Исомиддинов',
        'Шахзодхон Исхоки',
        'Шахзодчон Чонизоков',
        'Шахром Бобочонов',
        'Шерозхуча Шарифхучаев'
      ],
      selectedPeople: [],
      items: [],
      personItems: reactive({}),
      portions: {} // Хранение количества порций для каждого человека и блюда
    };
  },
  methods: {
    getPortion(person, item) {
      return this.portions[person] && this.portions[person][item.name] ? this.portions[person][item.name] : 1;
    },
    setPortion(person, item, event) {
      const value = Number(event.target.value);
      if (!this.portions[person]) {
        this.portions[person] = {};
      }
      this.portions[person][item.name] = value;
    },
    onPeopleSelect() {
      this.selectedPeople.forEach(person => {
        if (!this.personItems[person]) {
          this.personItems[person] = [];
        }
      });
    },
    toggleItem(person, item) {
      if (!this.personItems[person]) {
        this.personItems[person] = [];
      }

      const personItems = this.personItems[person];
      const index = personItems.findIndex(selectedItem => selectedItem.name === item.name);
      if (index === -1) {
        personItems.push(item);
      } else {
        personItems.splice(index, 1);
      }
    },
    isItemSelected(person, item) {
      if (!this.personItems[person]) return false;

      return this.personItems[person].some(selectedItem => selectedItem.name === item.name);
    },
    calculateTotal(person) {
      if (!this.personItems[person]) return 0;

      const itemsTotal = this.personItems[person].reduce((total, item) => {
        const portions = this.portions[person]?.[item.name] || 1;
        return total + (item.price * portions);
      }, 0);

      const share = this.calculateShare();
      return itemsTotal + share;
    },
    calculateOverallTotal() {
      const total = this.selectedPeople.reduce((overallTotal, person) => overallTotal + this.calculateTotal(person), 0);
      return total;
    },
    calculateShare() {
      const totalProductsCost = Object.values(this.costs).reduce((a, b) => a + b, 0);
      const total = totalProductsCost + this.serviceCharge;
      const share = this.selectedPeople.length ? total / this.selectedPeople.length : 0;
      return share;
    },
    calculateCosts() {
      const totalProductsCost = Object.values(this.costs).reduce((a, b) => a + b, 0);
      const total = totalProductsCost + this.serviceCharge;
      return total;
    },
    removePerson(person) {
      this.selectedPeople = this.selectedPeople.filter(p => p !== person);
      // this.$delete(this.personItems, person);
    },
    addItem() {
      if (this.newItem.name && this.newItem.price > 0) {
        this.items.push({...this.newItem});
        this.newItem.name = '';
        this.newItem.price = 0;
      }
    },
    updateCosts() {
      // Обновление стоимости продуктов и обслуживания
    },
    updatePersonItems() {
      for (const person of this.selectedPeople) {
        this.personItems[person] = this.selectedItems[person] || [];
      }
    },
    async generateImage() {

      const reportElement = document.querySelector("#report");

      if (reportElement && reportElement instanceof HTMLElement) {
        try {
          console.log('Generating image for element:', reportElement);

          html2canvas(reportElement, {
            backgroundColor: '#ffffff', // Установить фон
            scale: 2 // Масштаб для улучшения качества изображения
          }).then(canvas => {
            const dataUrl = canvas.toDataURL('image/png');
            const link = document.createElement('a');
            link.href = dataUrl;
            link.download = 'report.png';
            link.click(); // Инициировать скачивание изображения

            console.log('Image generated and download initiated');
          }).catch(error => {
            console.error('Ошибка при генерации изображения:', error);
          });
        } catch (error) {
          console.error('Не удалось создать изображение:', error);
        }
      } else {
        console.error('Элемент отчета не найден в DOM или не является элементом HTML.');
      }
    },
  },
  watch: {
    selectedPeople: 'onPeopleSelect',
    selectedItems: {
      handler() {
        this.updatePersonItems();
      },
      deep: true
    }
  }
};
</script>

<style scoped>.person-card:hover {
    transform: translateY(-1px);
    transition: transform .15s ease;
}

/* «Липкая» карточка итога справа */
.sticky-summary {
    position: sticky;
    top: 12px;
    z-index: 2;
}

/* Отчёт — аккуратная сетка */
.report-container {
    border-radius: 10px;
    overflow: hidden;
    border: 1px solid rgba(0, 0, 0, 0.08);
}

.report-header,
.report-row,
.report-footer {
    display: grid;
    grid-template-columns: 1.2fr 2.2fr 0.8fr;
    gap: 12px;
    align-items: center;
}

.report-header {
    background: #f5f5f5;
    font-weight: 600;
    padding: 10px 12px;
}

.report-header-cell {
    font-size: 0.85rem;
}

.report-body .report-row {
    padding: 10px 12px;
    border-top: 1px dashed rgba(0, 0, 0, 0.06);
}

.totals-row {
    background: #fafafa;
}

.report-footer {
    border-top: 1px solid rgba(0, 0, 0, 0.08);
    padding: 12px;
    background: #fff;
    font-weight: 600;
}

.report-cell {
    font-size: 0.9rem;
}

.report-name {
    display: flex;
    align-items: center;
    font-weight: 600;
}

.text-right {
    text-align: right;
}

.report-dishes {
    list-style: none;
    margin: 0;
    padding: 0;
}

.report-dish {
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    margin: 2px 0;
    line-height: 1.4;
}

.report-dish .dish-name {
    font-weight: 500;
}

.report-dish .dish-price,
.report-dish .dish-portion {
    opacity: 0.9;
}

.report-dish .times {
    margin: 0 6px;
    opacity: 0.7;
}

.report-dish .dot {
    width: 4px;
    height: 4px;
    border-radius: 50%;
    background: currentColor;
    opacity: 0.35;
    margin: 0 8px;
}

.report-dish.muted {
    opacity: 0.85;
    font-style: italic;
}

.stacked-costs > div {
    margin: 2px 0;
}

.portion-field {
    width: 80px;
}

/* Мелкие улучшения типографики */
.report-title {
    font-size: 1rem;
    font-weight: 600;
}
</style>
