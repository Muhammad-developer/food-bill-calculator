<template>
  <v-container>
    <!-- Форма добавления нового блюда -->
    <v-form @submit.prevent="addItem">
      <v-row dense>
        <v-col cols="6">
          <v-row style="padding: 0; margin: 4px;">
            <v-text-field
                v-model="newItem.name"
                label="Блюдо"
                required
                dense
                hide-details
                class="pa-0"
                style="font-size: 0.75rem; width: 100px;"
            ></v-text-field>
            <v-text-field
                v-model.number="newItem.price"
                label="Цена"
                type="number"
                required
                dense
                hide-details
                class="pa-0"
                style="font-size: 0.75rem; width: 100px;"
            ></v-text-field>
            <v-btn color="primary" type="submit" class="mt-1" small>Добавить</v-btn>
          </v-row>
        </v-col>
        <!-- Форма добавления стоимости продуктов и обслуживания -->
        <v-form @submit.prevent="updateCosts">
          <v-row dense style="padding: 2px; margin: 5px;">
            <v-text-field
                v-model.number="costs.bread"
                label="Хлеб"
                type="number"
                min="0"
                step="0.01"
                dense
                hide-details
                class="pa-0 bg-light-green-accent-1 rounded-lg m-2"
                style="font-size: 0.75rem;"
            ></v-text-field>
            <v-text-field
                v-model.number="costs.tea"
                label="Чай"
                type="number"
                min="0"
                step="0.01"
                dense
                hide-details
                class="pa-0 bg-light-green-accent-1 rounded-lg m-2"
                style="font-size: 0.75rem;"
            ></v-text-field>
            <v-text-field
                v-model.number="costs.cola"
                label="Кола"
                type="number"
                min="0"
                step="0.01"
                dense
                hide-details
                class="pa-0 bg-light-green-accent-1 rounded-lg m-2"
                style="font-size: 0.75rem;"
            ></v-text-field>
            <v-text-field
                v-model.number="serviceCharge"
                label="Обслуживание"
                type="number"
                min="0"
                step="0.01"
                dense
                hide-details
                class="pa-0 bg-green rounded-lg m-2"
                style="font-size: 0.75rem;"
            ></v-text-field>
            <v-btn color="primary" type="submit" class="mt-2 m-4 p-4" small>Обновить</v-btn>
          </v-row>
        </v-form>
      </v-row>
    </v-form>

    <!-- Выбор людей -->
    <v-row dense>
      <v-col cols="12">
        <v-autocomplete
            v-model="selectedPeople"
            :items="people"
            label="Люди"
            chips
            multiple
            clearable
            dense
            hide-details
            class="pa-0"
            style="font-size: 0.75rem;"
        ></v-autocomplete>
      </v-col>
    </v-row>

    <!-- Отображение блюд для выбранных людей -->
    <v-row dense>
      <v-col v-for="person in selectedPeople" :key="person" cols="12" sm="12" md="3">
        <v-card class="pa-2" elevation="1">
          <v-card-title class="text-h6" style="font-size: 0.875rem;">{{ person }}</v-card-title>
          <v-card-text>
            <v-list dense>
              <v-list-item-group v-for="item in items" :key="item.name" style="width: max-content; padding: 0;">
                <v-list-item-action style="padding: 0;">
                  <v-checkbox
                      @change="toggleItem(person, item)"
                      :input-value="isItemSelected(person, item)"
                      style="margin: 0;"
                  ></v-checkbox>
                  <v-list-item-title style="font-size: 1rem; padding: 5px; margin-top: -20px;">
                    {{ item.name }}
                  </v-list-item-title>
                  <v-list-item-subtitle style="font-size: 0.85rem; margin-top: -20px;">
                    {{ item.price }} смн.
                  </v-list-item-subtitle>
                  <v-text-field
                      v-if="isItemSelected(person, item)"
                      label="Порции"
                      type="number"
                      min="0"
                      step="0.5"
                      dense
                      hide-details
                      class="pa-0"
                      style="font-size: 0.75rem; width: 50px;"
                      @input="event => setPortion(person, item, event)"
                  ></v-text-field>
                </v-list-item-action>
              </v-list-item-group>
            </v-list>
          </v-card-text>
          <v-card-actions>
            <v-btn text @click="removePerson(person)" small>Удалить</v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>

    <!-- Общая сумма -->
    <v-row dense style="width: max-content;">
      <v-col>
        <v-card class="pa-2" elevation="1">
          <v-card-title style="font-size: 0.75rem;">Общая сумма</v-card-title>
          <v-card-subtitle style="font-size: 0.75rem;">{{ calculateOverallTotal().toFixed(2) }} смн.</v-card-subtitle>
        </v-card>
      </v-col>
    </v-row>

    <!-- Отчет (чек) -->
    <v-row dense>
      <v-col>
        <v-card class="pa-2" elevation="1">
          <v-card-title class="report-title">Отчет</v-card-title>
          <v-card-text ref="reportContent">
            <!-- Контейнер для отчета -->
            <div id="report" class="report-container">
              <div class="report-header">
                <div class="report-header-cell">Имя</div>
                <div class="report-header-cell">Блюда</div>
                <div class="report-header-cell">Сумма</div>
              </div>
              <div class="report-body">
                <div v-for="person in selectedPeople" :key="person" class="report-row">
                  <div class="report-cell">{{ person }}</div>
                  <div class="report-cell">
                    <ul>
                      <li v-for="item in personItems[person]" :key="item.name">
                        {{ item.name }} - {{ item.price }} смн. x {{ getPortion(person, item) }} порций
                      </li>
                        <li>
                            <span>Обслуживание и прочее: {{ calculateShare() }} смн.</span>
                        </li>
                    </ul>
                  </div>
                  <div class="report-cell">{{ calculateTotal(person).toFixed(2) }} смн.</div>
                </div>
                  <div class="report-row">
                      <div class="report-cell">
                      </div>
                      <div class="report-cell">
                          <span>Хлеб: {{ costs.bread }} смн.</span><br>
                          <span>Кола: {{ costs.cola }} смн.</span><br>
                          <span>Чай: {{ costs.tea }} смн.</span><br>
                          <span>Обслуживание: {{ serviceCharge }} смн.</span>
                      </div>
                      <div class="report-cell">
                          {{calculateCosts().toFixed(2)}} смн.
                      </div>
                  </div>
              </div>
              <div class="report-footer">
                <div class="report-cell footer-label">Общая сумма:</div>
                <div class="report-cell footer-value">{{ calculateOverallTotal().toFixed(2) }} смн.</div>
              </div>
            </div>
          </v-card-text>
          <v-card-actions>
            <v-btn color="primary" @click="generateImage" small>Сохранить как изображение</v-btn>
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

<style scoped>
.mini-card {
  width: max-content;
}

.v-card {
  margin-bottom: 4px;
  padding: 8px;
}

.v-list-item {
  cursor: pointer;
  font-size: 0.75rem;
}

.v-list-item-content {
  padding: 2px 4px;
}

.v-list-item-title, .v-list-item-subtitle {
  font-size: 0.75rem;
}

.v-btn {
  font-size: 0.75rem;
}

.v-text-field {
  font-size: 0.75rem;
}

.report-container {
  display: flex;
  flex-direction: column;
  width: 100%;
  background: #fff; /* Белый фон для контраста */
  border-radius: 4px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
}

.report-title {
  font-size: 0.875rem; /* Размер шрифта для заголовка */
  font-weight: bold;
  text-align: center;
  margin-bottom: 8px;
}

.report-header {
  display: flex;
  background: #f0f0f0;
  padding: 4px 8px;
  border-bottom: 1px solid #ddd;
  justify-content: space-between;
}

.report-header-cell {
  flex: 1;
  text-align: center;
  font-weight: bold;
  font-size: 0.75rem; /* Размер шрифта для заголовков */
}

.report-body {
  display: flex;
  flex-direction: column;
}

.report-row {
  display: flex;
  justify-content: space-between;
  padding: 4px 8px;
  border-bottom: 1px solid #ddd;
}

.report-cell {
  flex: 1;
  text-align: center;
  font-size: 0.75rem; /* Размер шрифта для ячеек */
}

.report-footer {
  display: flex;
  justify-content: space-between;
  padding: 4px 8px;
  border-top: 1px solid #ddd;
  background: #f0f0f0;
}

.footer-label {
  font-weight: bold;
}

.footer-value {
  font-weight: bold;
}
</style>
