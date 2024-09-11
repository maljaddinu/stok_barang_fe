<script>
import { ref } from 'vue';
import VueDatePicker from '@vuepic/vue-datepicker';
import '@vuepic/vue-datepicker/dist/main.css';
 import { Modal } from 'bootstrap'

const baseURL = "http://localhost:8000/api";
export default {

  name : 'App',
  components: {
    VueDatePicker
  },
  data() {
    return {
      modalData:null,
      listPengajuan:null, 
      listProduk:[],
      listKodeKaryawan: [],
      listKaryawan:null,
      selectedKaryawan:null,
      namaKaryawan:null, 
      departemenKaryawan:null,
      listRequestPengajuan:[],
      itemPengajuan:{
        kodeBarang:null,
        kuantiti:null, 
        keterangan:null,
      }

    }
  },
  methods : {
    async getDataKaryawan() {
       try {
        const res = await fetch(`${baseURL}/karyawan`);

        if (!res.ok) {
          const message = `An error has occured: ${res.status} - ${res.statusText}`;
          throw new Error(message);
        }

        const data = await res.json();

        const result = {
          status: res.status + "-" + res.statusText,
          headers: {
            "Content-Type": res.headers.get("Content-Type"),
            "Content-Length": res.headers.get("Content-Length"),
          },
          length: res.headers.get("Content-Length"),
          data: data,
        };

        // result.data.data.map(function(value,key){
        //   this.listKodeKaryawan.push(value.kode_karyawan);
        // });
        let arrayData = [];
        result.data.data.map(function(value,key){
          //console.log(value.kode_karyawan)
          arrayData.push(value.kode_karyawan);
        })
        this.listKodeKaryawan = arrayData;
        this.listKaryawan = result.data.data
      } catch (err) {
        //this.getResult = err.message;
      }
    },
    async getDataProduk() {
      try {
        const res = await fetch(`${baseURL}/produk`);

        if (!res.ok) {
          const message = `An error has occured: ${res.status} - ${res.statusText}`;
          throw new Error(message);
        }

        const data = await res.json();

        const result = {
          status: res.status + "-" + res.statusText,
          headers: {
            "Content-Type": res.headers.get("Content-Type"),
            "Content-Length": res.headers.get("Content-Length"),
          },
          length: res.headers.get("Content-Length"),
          data: data,
        };

        //this.listProduk = this.fortmatResponse(result);
        this.listProduk = result.data.data;
      } catch (err) {
        //this.getResult = err.message;
      }
    },
     async getPengajuanData() {
      try {
        const res = await fetch(`${baseURL}/pengajuan`);

        if (!res.ok) {
          const message = `An error has occured: ${res.status} - ${res.statusText}`;
          throw new Error(message);
        }

        const data = await res.json();

        const result = {
          status: res.status + "-" + res.statusText,
          headers: {
            "Content-Type": res.headers.get("Content-Type"),
            "Content-Length": res.headers.get("Content-Length"),
          },
          length: res.headers.get("Content-Length"),
          data: data,
        };

        //console.log(result.data.data);
        this.listPengajuan = result.data.data;
      } catch (err) {
        //this.getResult = err.message;
      }
    },
    tambahItem() {
        this.listRequestPengajuan.push(
          {
            kodeBarang:null,
            lokasi:null,
            tersedia:null,
            kuantiti:null,
            satuan:null, 
            keterangan:null,
            status:null
          }
        )
    },
    removeRow(key) {
      this.listRequestPengajuan.splice(key-1, 1);
    },
    getItemText(title) {
      return `${title.kode_sku} ${title.nama_produk}`;
    },
    checkValue(value) {
      value.tersedia = value.kodeBarang.total_stock - value.kodeBarang.total_out_stock;
      value.satuan = value.kodeBarang.satuan;
      value.lokasi = value.kodeBarang.kode_bin;
      this.calculate(value);
    },
    calculate(value)
    {
      var availableStock = value.kodeBarang.total_stock - value.kodeBarang.total_out_stock
      if(availableStock <= 0)
      {
        value.status = 0;
      }

      var qty = 0; 
      isNaN(parseFloat(value.kuantiti)) ? qty = 0 : qty = parseFloat(value.kuantiti);

      var checkCurrentAvailable = availableStock - qty;

      if(checkCurrentAvailable <= 0 && qty > 0)
      {
        value.status = 1
      }
      else if(checkCurrentAvailable >= 0 && qty > 0)
      {
        value.status = 2;
      }
      else if(qty == 0)
      {
        value.status = null
      }
    },
    async submit()
    {
      
      let flagError = false;
      let itemKosong = false;

      if(this.selectedKaryawan == null) {
        this.$swal({
          icon: "error",
          text: "Data Karyawan Kosong!"
        });
        return false; 
      }

      let listData = [];
      
      this.listRequestPengajuan.map((value)  => {
        if(value.kodeBarang.kode_sku == '' || value.kuantiti == 0 || value.status < 2) {
          flagError = true;

          if(value.status < 2)
          {
            itemKosong = true;
          }
          return;
        }

        listData.push({
          kodeBarang:value.kodeBarang.kode_sku,
          qty:value.kuantiti,
          keterangan:value.keterangan
        })
        
      })

      if(flagError)
      {
        if(itemKosong)
        {
            this.$swal({
              icon: "error",
              text: "Terdapat SKU yang Tidak Tersedia!"
            });
            itemKosong = false; 

        }
        else 
        {
          this.$swal({
            icon: "error",
            text: "Kode SKU atau Kuantiti kosong!"
          });
        }
        flagError = false; 
        return false; 
      }

      let formData =   
      {
        kodeKaryawan : this.selectedKaryawan,
        listPengajuan : listData 
      }


       const requestOptions = {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(formData)
      };
      
      try {
        const res = await fetch(`${baseURL}/pengajuan`, requestOptions)
          
        const result = await res.json();
        if(result.success) {
          this.$swal({
            icon: "success",
            text: result.message
          });
        } else {
           this.$swal({
            icon: "error",
            text: result.message
          });
        }

        this.modalData.hide();
        this.getPengajuanData();
        this.getDataKaryawan();
        this.getDataProduk();
      } 
      catch (error)
      {
        console.log(error)
      }
    
      
    },
    tambahPengajuan()
    {
      this.selectedKaryawan = null,
      this.namaKaryawan = null, 
      this.departemenKaryawan = null,
      this.listRequestPengajuan = [];
      this.modalData.show();
    }
  },
  mounted() {
    this.modalData = new Modal(document.getElementById('staticBackdrop'));
    this.getPengajuanData();
    this.getDataKaryawan();
    this.getDataProduk();
  },
  watch: {
    selectedKaryawan(newValue) {
      this.listKaryawan.map((value,key) => {
        if(value.kode_karyawan == newValue)
        {
          this.namaKaryawan = value.nama_karyawan
          this.departemenKaryawan = value.divisi
        }
      })
    }
  }

}

const date = ref(null);
</script>

<template>
  <div class="container-fluid">
    <div class="row">
      <div class="col-sm-12">
        <button type="button" class="btn btn-primary" @click="tambahPengajuan()">Tambah Pengajuan</button>    
      </div>
    </div>
    <div class="row">
      <div class="col-sm-12">
        <table class="table table-striped">
          <thead>
            <tr>
              <th scope="col">#</th>
              <th scope="col">Kode Pengajuan</th>
              <th scope="col">Nama Karyawan</th>
              <th scope="col">Divisi</th>
              <th scope="col">Tanggal Pengajuan</th>
              <th scope="col">Item</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(value, key) in listPengajuan">
              <th scope="row">{{++key}}</th>
              <td>{{value.head.kode_pengajuan}}</td>
              <td>{{value.head.nama_karyawan}} / {{value.head.kode_karyawan}}</td>
              <td>{{value.head.divisi}}</td>
              <td>{{value.head.tanggal_pengajuan}}</td>
              <td>
                <table class="table">
                  <thead>
                    <tr>
                      <th scope="col">#</th>
                      <th scope="col">SKU</th>
                      <th scope="col">Produk</th>
                      <th scope="col">Satuan</th>
                      <th scope="col">Total Pengajuan</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(itemValue, itemKey) in value.item">
                      <th scope="row">{{++itemKey}}</th>
                      <td>{{itemValue.kode_sku}}</td>
                      <td>{{itemValue.nama_produk}}</td>
                      <td>{{itemValue.satuan}}</td>
                      <td>{{itemValue.total}}</td>
                    </tr>
                  </tbody>
                </table>
              </td>
            </tr>
          </tbody>
        </table>    
      </div>
    </div>
  </div>    
  <div class="modal fade" id="staticBackdrop" data-bs-backdrop="static" data-bs-keyboard="false" tabindex="-1" aria-labelledby="staticBackdropLabel" aria-hidden="true">
  <div class="modal-dialog modal-dialog-centered modal-xl">
    <div class="modal-content">
      <div class="modal-header">
        <h1 class="modal-title fs-5" id="staticBackdropLabel">Tambah Data Pengajuan</h1>
        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
      </div>
      <div class="modal-body">
        <div class="container">
          <div class="row">
            <div class="col-4">
              <div class="mb-3">
                <label for="nikPeminta" class="form-label">NIK Peminta</label>
                <v-autocomplete
                  :items="this.listKodeKaryawan"
                  variant="outlined"
                  v-model="selectedKaryawan"
                ></v-autocomplete>
              </div>
            </div>
            <div class="col-4">
              <div class="mb-3">
                <label for="namaPeminta" class="form-label">Nama Peminta</label>
                <v-text-field id="namaPeminta" variant="outlined" readonly v-model="namaKaryawan"></v-text-field>
              </div>
            </div>
            <div class="col-4">
             <div class="mb-3">
                <label for="departemenPeminta" class="form-label">Departemen</label>
                <v-text-field id="departemenPeminta" variant="outlined" readonly v-model="departemenKaryawan"></v-text-field>
              </div>
            </div>
          </div>
          <div class="row">
            <div class="col-sm-12">
              <button type="button" class="btn btn-success" @click="tambahItem">Tambah Item</button>    
            </div>
          </div>
          <div class="row">
            <table class="table table-bordered table-stripped">
              <thead>
                  <tr>
                      <th style="width:5%">#</th>
                      <th style="width:20%">Barang</th>
                      <th style="width:5%">Lokasi</th>
                      <th style="width:5%">Tersedia</th>
                      <th style="width:10%">Kuantiti</th>
                      <th style="width:5%">Satuan</th>
                      <th style="width:20%">Keterangan</th>
                      <th style="width:10%">Status</th>
                      <th style="width:5%">*</th>
                  </tr>
              </thead>
              <tbody>
                  <tr v-for="(value, key) in listRequestPengajuan">
                  <th scope="row">{{++key}}</th>
                      <td>
                        <v-autocomplete
                          :items="this.listProduk"
                          variant="outlined"
                          :item-title="getItemText"
                          item-value="kode_sku"
                          v-model="value.kodeBarang"
                          @update:modelValue="checkValue(value)"
                          return-object
                        ></v-autocomplete>
                      </td>
                      <td>{{value.lokasi == null ? '-' : value.lokasi}}</td>
                      <td>{{value.tersedia == null ? '-' : value.tersedia}}</td>
                      <td><input type="number" @input="calculate(value)" v-model="value.kuantiti" class="form-control"></td>
                      <td>
                         {{value.satuan == null ? '-' : value.satuan}}
                      </td>
                      <td>
                          <input type="text" v-model="value.keterangan" class="form-control">
                      </td>
                      <td>
                        <p class="text-black" v-if="value.status == null"> - </p>
                        <p class="text-danger" v-else-if="value.status == 0"> Kosong </p>
                        <p class="text-warning" v-else-if="value.status == 1"> Sebagian </p>
                        <p class="text-success" v-else> Terpenuhi </p>
                      </td>
                      <td>
                          <button type="button" class="btn btn-danger" @click="removeRow(key)">X</button>
                      </td>
                  </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-primary" @click="submit()">simpan</button>
      </div>
    </div>
  </div>
</div>
</template>

