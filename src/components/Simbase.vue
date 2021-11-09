<template>
  <div>
    <b-table class="table-borderless" :items="items" :fields="fields" >
      <template v-slot:cell(status)="row" >
        <b-dropdown :style="'margin: 0 !important;background-color:#'+row.item.statusColor" class="m-2 color-dropdown">
          <b-dropdown-item v-for="color in colors" :key="color.d_id" :value="row.item.status" @click="row.item.status = color.d_id;saveData(row.item)">
            <div @click="onColorPick(color, row.item)" :data-color="color.d_color_code" :style="'width: 25px;height: 25px;background-color: #' + color.d_color_code "></div>
          </b-dropdown-item>
        </b-dropdown>
      </template>
      <template v-slot:cell(accommodation)="row">
        <b-form-select class="form-control" v-model="row.item.accommodation" :options="optionsAccommodation" @change="saveData(row.item)"></b-form-select>
      </template>
      <template v-slot:cell(number_of_rooms)="row">
        <b-form-input
          @keyup="saveData(row.item)"
          @change="saveData(row.item);getNumbeOfRooms()"
          type="number"
          v-model="row.item.number_of_rooms"
        />
      </template>
      <template v-slot:cell(room_type)="row">
        <b-form-select class="form-control" v-model="row.item.room_type" :options="optionsRoomType" @change="saveData(row.item)"></b-form-select>
      </template>
      <template v-slot:cell(checkin_date)="row">
        <b-form-datepicker
          :date-format-options="{ day: 'numeric', month: 'numeric', year: 'numeric'  }"
          @input="saveData(row.item)"
          v-model="row.item.checkin_date"
        />
      </template>
      <template v-slot:cell(nts)="row">
        <b-form-input
          @keyup="saveData(row.item)"
          @change="saveData(row.item)"
          type="number"
          v-model="row.item.nts"
        />
      </template>
      <template v-slot:cell(price)="row">
        <b-form-input
          type="number"
          v-model="row.item.price"
        />
      </template>
      <template v-slot:cell(cost)="row">
        <b-form-input
          type="number"
          v-model="row.item.cost"
        />
      </template>
      <template #cell(actions)="row">
        <b-button @click="deleteRow(row.item, row.index, $event.target)" class="delete-btn">
          X
        </b-button>
      </template>

      <template v-slot:custom-foot>
            <tr class="footer-tr">
              <td>
                <b-button @click="addRow()" class="add-btn">ADD</b-button>       
              </td>
              <td></td>
              <td>{{ numberOfRooms }}</td>
              <td></td>
              <td></td>
              <td></td>
              <td>{{price}} €</td>
              <td>{{cost}} €</td>
              <td></td>
            </tr>
            <tr class="footer-tr">
              <td></td>
              <td></td>
              <td></td>
              <td></td>
              <td></td>
              <td></td>
              <td>TOTAL GP {{ totalGP}} €</td>
              <td></td>
              <td></td>
            </tr>
          </template>
    </b-table>
  </div>
</template>

<style>
body {
  padding: 1rem;
  background-color: #f7e7f9 !important;
}
.btn:focus, .btn:active, {
  border: none !important;
  outline: 0 !important;
}
.delete-btn {
  vertical-align: -webkit-baseline-middle !important;
  padding: 0 !important;
  width: 20px !important;
  height: 20px !important;
  border-radius: 50% !important;
  font-size: 14px !important;
  font-weight: bold !important;
  background-color: #0a7aff !important;
  border: none !important;
  outline: none !important;
}
.add-btn {
  background-color: #007fff !important;
  border:none !important;
  outline: none !important;
}
.footer-tr {
  font-weight: bold;
}
.color-dropdown .dropdown-toggle {
  background-color: unset !important;
  border: none;
  width: 35px;
  height: 35px;
  vertical-align: middle;
}
.color-dropdown .dropdown-toggle:after {
  display: none;
}
.color-dropdown ul {
  width: 100% !important;
  min-width: unset !important;
}
.color-dropdown ul li  {
  padding: 3.5px;
}
.color-dropdown ul li a {
  width: 25px;
  height: 25px;
  position: relative;
  padding: 0;
}
.color-dropdown ul li a div {
    top: 0;
    left: 0;
    position: absolute;
}

</style>

<script>

import axios from "axios";
export default {
  data() {
    return {
      fields: [
        { key: 'status', label: 'Status', thStyle:{ width: '5%'} },
        { key: 'accommodation', label: 'Accommodation', thStyle:{ width: '20%'} },
        { key: 'number_of_rooms', label: 'Number of rooms', thStyle:{ width: '10%'} },
        { key: 'room_type', label: 'ROOM TYPE', thStyle:{ width: '15%'} },
        { key: 'checkin_date', label: 'CHECKIN-DATE', thStyle:{ width: '15%'} },
        { key: 'nts', label: 'NTS', thStyle:{ width: '10%'} },
        { key: 'price', label: 'PRICE €', thStyle:{ width: '10%'} },
        { key: 'cost', label: 'COST €', thStyle:{ width: '10%'} },
        { key: 'actions', label: '', thStyle:{ width: '5%'} },
      ],
      optionsAccommodation: [],
      optionsRoomType:  [],
      colors:[],
      selectedColor:'',
      defaultColor:'',
      defaultColorStatus:'',
      numberOfRooms:0,
      price:'0,00',
      cost:'0,00',
      totalGP:'0,00',
      items: []
    };
  },
  mounted() {
    this.getData();
  },
  methods: {
    deleteRow(item){
      const index = this.items.findIndex(s => s === item)
      this.items.splice(index, 1);
      this.getSumValues();
      this.getNumbeOfRooms();
    },
    addRow(){
      this.items.push({ statusColor:this.defaultColor, status: this.defaultColorStatus, accommodation: null, number_of_rooms:0, room_type:null, checkin_date:null, nts:0, price:0, cost:0});
    },
    getData(){
      axios
      .get('http://localhost:8080/api/accommodations-data')
      .then(response => {
        this.optionsAccommodation = response.data['Accommodation suppliers'].map(function(item){
          return {value: item.o_id, text: item.o_name};
        })
        this.optionsRoomType = response.data['dic_room_type'].map(function(item){
          return {value: item.d_id, text: item.d_name};
        })
        this.optionsAccommodation.unshift({value:null, text:'Select accommodation'});
        this.optionsRoomType.unshift({value:null, text:'Select room type'});

        this.colors        = response.data.dic_status;
        this.selectedColor = response.data.dic_status[0].d_color_code;
        this.defaultColor  = response.data.dic_status[0].d_color_code;
        this.defaultColorStatus  = response.data.dic_status[0].d_id;

        this.items.push({ statusColor:this.defaultColor, status: this.defaultColorStatus, accommodation: null, number_of_rooms:0, room_type:null, checkin_date:null, nts:0, price:0, cost:0});

      });
    },
    onColorPick(color, row) {
      this.selectedColor = color.d_color_code;
      row.statusColor    = color.d_color_code;
    },
    saveData(row){
      if (row.status && row.accommodation && row.number_of_rooms != 0 && row.room_type && row.checkin_date && row.nts != 0) {
        axios
        .post(process.env.API_URL+'http://localhost:8080/api/save-accommodations-data', row)
        .then(response => {
          if (response.data.price != null) {
            row.price = response.data.price;
          }else {
            row.price = 0;
          }
          if (response.data.cost != null) {
            row.cost  = response.data.cost;
          }else {
            row.cost  = 0;
          }
          this.getSumValues();
        });
      }
    },
    getSumValues(){    
      var price  = 0;
      var cost   = 0;
      for (var i = 0; i < this.items.length; i++) {
        price  += parseInt(this.items[i].price);
        cost   += parseInt(this.items[i].cost);
      }
      this.price         = price;
      this.cost          = cost;
      this.totalGP       = price+cost;
    },
    getNumbeOfRooms(){
      var number = 0;
      for (var i = 0; i < this.items.length; i++) {
        number += parseInt(this.items[i].number_of_rooms);
      }
      this.numberOfRooms = number;
    }
  }
};
</script>

