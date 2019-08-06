<template>
  <v-content>
    <v-layout
      text-center
      wrap
    >
      <v-flex xs4>
        <v-card>
          <v-card-title primary-title>
            <div>
              <h3>Resumo Por Semana</h3>
              <div
                v-for="(item, index) in weeks"
                :key="index"
              >
              <v-checkbox
                :label="`Semana ${item.week} =(${item.horas.toFixed(2)}) R$ ${item.money.toFixed(2)}`"
                :value="item.money"
                ref="selectedItems"
                @click="totalAReceber"
                ></v-checkbox>
              
              <!-- - {{item.horas.toFixed(2)}} -->
              </div>
              <hr>
              <div>
                Total: R$ {{total}}
              </div>
            </div>
          </v-card-title>
        </v-card>
      </v-flex>
      <v-flex xs6 ml-4>
        <v-card>
          <v-card-title primary-title>
            <div>
              <v-date-picker
                light
                v-model="temp.date"
                scrollable
                show-week
                ></v-date-picker>
              <v-time-picker 
                light
                header-color="success"
                v-model="temp.entrada"
                :max="temp.saida"
                scrollable
                mr-2
                format="24hr"
              ></v-time-picker>
              <v-time-picker 
                light
                header-color="warning"
                v-model="temp.saida"
                :min="temp.entrada"
                scrollable
                format="24hr"></v-time-picker>
            </div>
          </v-card-title>
          <v-card-actions>
            <v-btn
            @click="saveTemp"
            color="success">Salvar</v-btn>
          </v-card-actions>
        </v-card>
        
      </v-flex>
      <v-flex xs12 mt-5>
        <v-card>
          <v-card-title primary-title>
            <v-layout row wrap>
              <v-flex
                xs12
              >
              <v-simple-table>
                <thead>
                  <tr>
                    <th class="text-left">Data</th>
                    <th class="text-left">Entrada</th>
                    <th class="text-left">Saida</th>
                    <th class="text-left">Extras</th>
                    <th class="text-left">$</th>
                  </tr>
                </thead>
                <tbody>
                  <tr
                    v-for="(item, index) in horarios"
                    :key="index"
                  >
                    <td class="text-left">{{item.date}}</td>
                    <td class="text-left">{{getHours(item.entrada)}}</td>
                    <td class="text-left">{{getHours(item.saida)}}</td>
                    <td class="text-left">{{getDiff(item.entrada,item.saida)}}</td>
                    <td class="text-left">{{getMoneys(item.entrada,item.saida)}}</td>
                    <td>
                      <v-btn 
                        color="success"
                        @click="editItem(index)"
                        mr-2
                      >
                        <v-icon>mdi-pencil</v-icon>
                      </v-btn>
                      <v-btn
                        color="warning"
                        @click="deleteItem(index)"
                      >
                        <v-icon>mdi-trash-can-outline</v-icon>
                      </v-btn>
                    </td>
                  </tr>
                </tbody>
              </v-simple-table>
              </v-flex>
            </v-layout>
          </v-card-title>
        </v-card>
      </v-flex>
    </v-layout>
  </v-content>
</template>

<script>
import moment from 'moment'
export default {
  data: () => ({
    temp:{
      date:'',
      entrada:'08:30',
      intervalo:'00:00',
      saida:'18:30',
      id:false,
    },
    total:0,
    horarios:[],
    weeks:[]
  }),
  computed:{

  },
  mounted(){
    var local = localStorage.getItem('horas');
    if(local != undefined)
    {
      this.horarios = JSON.parse(local);
    }
    this.temp.date = moment().format("YYYY-MM-DD")

    this.setWeeks();
  },
  methods: {
    getHours(temp){
      return moment(temp).format('HH:mm')
    },
    getDiff(temp1,temp2){
      temp1 = moment(temp1)
      temp2 = moment(temp2)

      var duration = moment.duration(temp2.diff(temp1));
      if(duration.hours() >= 7)
      {
        duration = duration.subtract(7,'hours');
        if(duration.minutes() > 30)
          duration = duration.subtract(30,'minutes');
      }
      return `${duration.hours()}:${duration.minutes()}`;
    },
    getMoneys(temp1,temp2){
      temp1 = moment(temp1)
      temp2 = moment(temp2)

      var duration = moment.duration(temp2.diff(temp1));
      if(duration.hours() >= 7)
      {
        duration = duration.subtract(7,'hours');
        if(duration.minutes() > 30)
          duration = duration.subtract(30,'minutes');
      }
      var horas = (duration.asHours());

      return `R$ ${((horas)*12.5).toFixed(2)}`
    },
    editItem(key)
    { 
      var item = this.horarios[key];
      var entrada = moment(item.entrada);
      var saida = moment(item.saida);
      var date = moment(item.date,'DD/MM/YYYY');
      this.temp.id = key;
      this.temp.date = date.format("YYYY-MM-DD");
      this.temp.entrada = entrada.format("HH:mm")
      this.temp.saida = saida.format("HH:mm")
    },
    deleteItem(key){
      this.horarios = this.horarios.filter((item,index) => {
        return index != key
      })
      this.saveLocal();
    },
    saveLocal()
    {
      localStorage.setItem('horas',JSON.stringify(this.horarios));
      this.setWeeks();
    },
    setWeeks()
    {
      this.weeks = []
      this.horarios.forEach(item => {
        var temp1 = moment(item.entrada);
        var getWeek = moment(item.date,'DD/MM/YYYY').format('w');
        
        var temp2 = moment(item.saida)

        var duration = moment.duration(temp2.diff(temp1));

        if(duration.hours() >= 7)
          duration = duration.subtract(7,'hours');
        if(duration.hours() >= 7 && duration.minutes() > 30)
          duration = duration.subtract('30','minutes');
        var horas = duration.asHours();
        var money = horas*12.5;
        var index = this.weeks.findIndex(item => item.week == getWeek);

        if(index >= 0)
        {
          // horas 
          // money
          var week = this.weeks[index];
          week.horas+=horas;
          week.money+=money;
          this.weeks[index] = week;
        }
        else
        {
          this.weeks.push({
            week:getWeek,
            horas,
            money,
          })
        }
      })
    },
    saveTemp()
    {
      var splitEntrada = this.temp.entrada.toString().split(":");
      var entrada = moment();
      var splitSaida = this.temp.saida.toString().split(":");
      var saida = moment();

      entrada.set('hour',splitEntrada[0])
      entrada.set('minute',splitEntrada[1])
      entrada.set('date',this.temp.date)

      saida.set('hour',splitSaida[0])
      saida.set('minute',splitSaida[1])
      saida.set('date',this.temp.date)

      // entrada = parseInt(entrada.format('X'))
      // saida = parseInt(saida.format('X'))

      var date = moment(this.temp.date,'YYYY-MM-DD');


      if(this.temp.id !== false)
      {
        this.horarios[this.temp.id] = {
          date:date.format("DD/MM/YYYY"),
          entrada,
          saida,
        }
      }
      else
      {
        this.horarios.push({
          date:date.format("DD/MM/YYYY"),
          entrada,
          saida,
        });
      }

      this.temp = {
        date:moment().format("YYYY-MM-DD"),
        entrada:'08:30',
        intervalo:'00:00',
        saida:'18:30',
        id:false,
      }
      this.saveLocal()
    },
    totalAReceber()
    {
      var total = 0;
      this.$refs.selectedItems.forEach(item => {
        var input = item.$refs.input;
        if(input.checked)
        {
          total+=parseFloat(input.value)
        }
      })
      this.total = total.toFixed(2);
    }
  },
};
</script>
