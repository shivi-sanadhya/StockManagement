<template>
    <span>
        <div class="row">
            <div class="col-md-4">
                <div class="form-group">
                    <label> Start Date*</label>
                        <client-only>
                            <date-picker
                                input-class="form-control col-md-12"
                                class="datepicker"
                                format="d-MM-yyyy"
                                v-model="startDate"
                            ></date-picker>
                        </client-only>
                </div>
            </div>                                                                                                
            <div class="col-md-4">
                <div class="form-group">
                    <label> End Date*</label>
                        <client-only>
                          <date-picker
                            input-class="form-control col-md-12"
                            class="datepicker"
                            format="d-MM-yyyy"
                            v-model="selectedDate"
                          ></date-picker>
                        </client-only>
                </div>
            </div>
            <div class="col-md-4">
                <div class="form-group">
                    <button class="btn btn-primary" @click="search()" type="submit">
                         <i class="fa fa-search"></i>
                          Search
                    </button>
                </div>
            </div>         
        </div>
        <div class="row">
            <div class="col-md-4">
                <div class="form-group">
                    <label >Item Name*</label>
                    <select2-multiple-control :options="getitemData" v-model="itemBind"  />
                </div>
            </div>  
            
            <div class="col-md-4">
                <div class="form-group">
                    <label >Branch Name*</label>
                    <select2-multiple-control :options="getbranchData" v-model="branchBind"  />
                </div>
            </div>
           <div class="col-md-4">
                <div class="form-group">
                    <button class="btn btn-primary" @click="getDetails()" type="submit">
                         <i class="fa fa-repeat"></i>
                          Reset
                    </button>
                </div>
            </div> 
        </div>
            <div class="row">
                <div class="col-md-12 col-12" >
                    <div class="table-responsive">
                    <table class="table table-striped dataex-html5-selectors dataTable">    
                <thead>
                    <tr>
                        <th>Item Name</th>
                        <th v-if="mode=='sale'">Sales Quantity</th>
                        <th v-else>Purchase Quantity</th>
                        <th>Branch</th>
                    </tr>
                     </thead>
                    <tbody> 
                    <tr v-for="(object,index) in fetchedObjects" :key="index">
                        <td >{{ object.name }}</td>
                        <td >{{ object.Quantity }}</td>
                        <td >{{ object.branchName }}</td>
                    </tr>
                    </tbody>           
             </table>
            </div>                 
                </div>
         </div>
        
    </span>
</template>
<script>
import axios from 'axios' 
import select2 from '../components/select2Component'
import Select2MultipleControl from 'v-select2-multiple-component'

export default {
     components:{
        select2,
        Select2MultipleControl
    },
      props: {
        id: {
            type: String,
            required: false
        },
        mode:{
            type: String,
            required: true
        },
        value: {
            type: Number,
            required: false
        }
    },
     data(){
        return{
            getitemData: [],
            getbranchData: [],
            selectedItems:[],
            fetchedObjects:"",
            // id:"",
            myBranch:"",
            itemBind:[],
            branchBind:[],
            // startDate: new Date(),
            selectedDate: new Date(),
             allItems:[],
             allbranch:[],
             reportObject:""
        }
    },  
    watch: {
           
        },
     computed:{
        startDate(){
            let dt = new Date();
            dt.setMonth(dt.getMonth()-1);
            return dt;
        }
     },
     created(){
            this.getDetails();        
    },
    filters:{
        filterDateFormat(val){
            let date = new Date(val);
            date.setHours(date.getHours()+6);
            return date.getDate() + '-' + (date.getMonth()+1) + '-' + date.getFullYear() ;
        }
    },
    methods:{
        changeDateFormat(dt){
            return dt.getFullYear()+'-'+(dt.getMonth()+1)+'-'+dt.getDate();
        },
        getDetails(){
            axios.get('http://localhost:4000/item/')
                .then(res=>{
                this.allItems=res.data;
                const options=[]
                for(let index in this.allItems){
                    options.push({
                        "id": this.allItems[index].itemId,
                        "text": this.allItems[index].name
                    });
                    //options.push(this.allItems[index].itemId)
                }
                this.getitemData= options;
            });
            

            axios.get('http://localhost:4000/branch/')
                .then(res=>{
                this.allbranch=res.data;
                const options=[]
                for(let index in this.allbranch){
                    options.push({
                        "id": this.allbranch[index].branchId,
                        "text": this.allbranch[index].branchName
                    });
                    //options.push(this.allItems[index].itemId)
                }
                this.getbranchData= options;
                });
            
            this.itemBind=[]
             this.branchBind=[]
            if(this.mode=="sale")
            {   
                axios.get('http://localhost:4000/report/'+this.changeDateFormat(this.startDate)+'/'+this.changeDateFormat(this.selectedDate))
                .then(response => {
                    this.fetchedObjects = response.data;
                    // console.log(this.fetchedObjects);
                });
            }
            else if(this.mode=="purchase")
            {
                axios.get('http://localhost:4000/purchasereport/'+this.changeDateFormat(this.startDate)+'/'+this.changeDateFormat(this.selectedDate))
                .then(response => {
                    this.fetchedObjects = response.data;
                    // console.log(this.fetchedObjects);
                });
            }
        },
        search(){
            if(this.mode=="sale"){
                if(this.itemBind.length>0 && this.branchBind.length>0)
                {
                    axios.get('http://localhost:4000/report/'+this.changeDateFormat(this.startDate)+'/'+this.changeDateFormat(this.selectedDate))
                    .then(response => {
                        this.reportObject = response.data;
                        this.fetchedObjects = this.reportObject.filter(object=>{    
                            for(let items of this.itemBind){
                                if(object.itemId == items){
                                    return true;
                                }
                            }

                            for(let branch of this.branchBind){
                                if(object.fkBranchId == branch){
                                    return true;
                                }
                            }
                        });
                    });
                }
                else if(this.itemBind.length>0){
                 axios.get('http://localhost:4000/report/'+this.changeDateFormat(this.startDate)+'/'+this.changeDateFormat(this.selectedDate))
                    .then(response => {
                        this.reportObject = response.data;
                        this.fetchedObjects = this.reportObject.filter(object=>{    
                            for(let items of this.itemBind){
                                if(object.itemId == items){
                                    return true;
                                }
                            }
                        });
                    });
                }
                else if(this.branchBind.length>0){
                 axios.get('http://localhost:4000/report/'+this.changeDateFormat(this.startDate)+'/'+this.changeDateFormat(this.selectedDate))
                    .then(response => {
                        this.reportObject = response.data;
                        this.fetchedObjects = this.reportObject.filter(object=>{   
                            for(let branch of this.branchBind){
                                if(object.fkBranchId == branch){
                                    return true;
                                }
                            }
                        });
                    });
                }
                else{
                     axios.get('http://localhost:4000/report/'+this.changeDateFormat(this.startDate)+'/'+this.changeDateFormat(this.selectedDate))
                .then(response => {
                    this.fetchedObjects = response.data;
                    // console.log(this.fetchedObjects);
                });
                }
            }
            else{
                if(this.itemBind.length>0 && this.branchBind.length>0)
                {
                    axios.get('http://localhost:4000/purchasereport/'+this.changeDateFormat(this.startDate)+'/'+this.changeDateFormat(this.selectedDate))
                    .then(response => {
                        this.reportObject = response.data;
                        this.fetchedObjects = this.reportObject.filter(object=>{    
                            for(let items of this.itemBind){
                                if(object.itemId == items){
                                    return true;
                                }
                            }

                            for(let branch of this.branchBind){
                                if(object.fkBranchId == branch){
                                    return true;
                                }
                            }
                        });
                    });
                }
                else if(this.itemBind.length>0){
                    axios.get('http://localhost:4000/purchasereport/'+this.changeDateFormat(this.startDate)+'/'+this.changeDateFormat(this.selectedDate))
                        .then(response => {
                            this.reportObject = response.data;
                            this.fetchedObjects = this.reportObject.filter(object=>{    
                                 console.log(object.itemId); 
                                for(let items of this.itemBind){
                                    console.log(items);
                                    if(object.itemId == items){
                                        return true;
                                    }
                                }
                            });
                        });
                    }
                else if(this.branchBind.length>0){
                    axios.get('http://localhost:4000/purchasereport/'+this.changeDateFormat(this.startDate)+'/'+this.changeDateFormat(this.selectedDate))
                        .then(response => {
                            this.reportObject = response.data;
                            this.fetchedObjects = this.reportObject.filter(object=>{   
                                console.log(object.fkBranchId); 
                                for(let branch of this.branchBind){
                                    console.log(branch);
                                    if(object.fkBranchId == branch){
                                        return true;
                                    }
                                }
                            });
                        });
                  }
                  else{
                       axios.get('http://localhost:4000/purchasereport/'+this.changeDateFormat(this.startDate)+'/'+this.changeDateFormat(this.selectedDate))
                .then(response => {
                    this.fetchedObjects = response.data;
                    // console.log(this.fetchedObjects);
                });
                  }        
            }
        }
    }
 }
</script>