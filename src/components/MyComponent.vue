
<template>
    <div class="header-search">
        <input type="text" class="search-input" v-model="searchName" placeholder="Search country..."/>
        <div>
            Sort Name By :
            <select v-model="sortType">
                <option value="asc">ASC</option>
                <option value="desc">DESC</option>
            </select>
        </div>
        <div>Total page : {{ totalPages }}</div>
    </div>

    <table>
        <thead>
            <tr>
                <th>No</th>
                <th>Flag</th>
                <th>Country Name</th>
                <th>CCA2</th>
                <th>CCA3</th>
                <th>Native Name</th>
                <th>Alternative Name</th>
                <th>IDD</th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="(country , index) in paginatedData" :key="index">
                <td>{{ index + 1 }}</td>
                <td>
                    <div class="display-img">
                        <img :src="country.flags.png" />
                    </div>
                </td>
                <td @click="showInfo(country)">{{ country.name.official }}</td>
                <td>{{ country.cca2 }}</td>
                <td>{{ country.cca3 }}</td>
                <td>
                    <ul>
                        <li v-for="(outerValue, outerKey) in country.name.nativeName" :key="outerKey">
                            <strong>{{ outerKey }}</strong>:
                            <ul>
                            <li v-for="(innerValue, innerKey) in outerValue" :key="innerKey">
                                {{ innerKey }}: {{ innerValue }}
                            </li>
                            </ul>
                        </li>
                    </ul>
                </td>
                <td>
                    <ul>
                        <li v-for="(item,id) in country.altSpellings" :key="id">{{ item }}</li>
                    </ul>
                </td>
                <td>
                    {{ country.idd.root }}
                    <ul>
                        <li v-for="(suffix,id) in country.idd.suffixes" :key="id">{{ suffix  }}</li>
                    </ul>
                </td>
            </tr>
        </tbody>
    </table>

    <!-- pagination -->
    <div class="pagination-container">
        <button @click="prePage()">pre</button>
        <div> {{ currentPage }}/{{ totalPages }} </div>
        <button @click="nextPage()">Next</button>
    </div>

    <!-- modal section -->
    <div  id="myModal" class="modal">
        <div class="modal-content">
            <span class="close" @click="closeModal()">&times;</span>
            <ul v-for="(data,key) in object" :key="key">
                <li v-if="!usedKey.includes(key)">
                        <strong>{{ key }}</strong>:
                        <div v-if="typeof data == 'object'">
                            <ul v-for="(subdata, subkey) in data" :key="subkey">
                                <li v-if="!usedKey.includes(`${key}.${subkey}`)">
                                    <strong v-if="isNaN(subkey)">{{ subkey }} : </strong>
                                    
                                    <ul v-if="typeof subdata == 'object'" class="smalldata-container" >
                                        <li class="smalldata" v-for="(smalldata, smallkey) in subdata" :key="smallkey">
                                            <strong v-if="isNaN(smallkey)">{{ smallkey }} :</strong>
                                             {{ smalldata }}
                                        </li>
                                    </ul>
                                    <span v-else>
                                        <img v-if="subkey == 'png' || subkey == 'svg'" :src="subdata" alt="" style="width: 100px;">
                                        <a v-else-if="subkey == 'googleMaps' || subkey == 'openStreetMaps'" :href="subdata" target="_blank">{{ subdata }}</a>
                                        <span v-else>{{ subdata }}</span>
                                    </span>
                                </li>
                            </ul>
                        </div>
                        <span v-else>{{ data }}</span>
                </li>
            </ul>
        </div>
    </div>
</template>
<script setup>
import axios from 'axios';
import {ref, onMounted, computed, watch} from "vue"

const rowPerPage = 25
const currentPage = ref(1)
const searchName = ref('')
var countries = ref([])
const sortType = ref('')

const object = ref({})

const usedKey = ref(['name.official','name.nativeName','cca2','cca3','altSpellings','idd','flags.png'])

onMounted(() => {
    fetchApi()
})
const fetchApi = async () => {
    await axios.get('https://restcountries.com/v3.1/all')
                .then(response => {
                    countries.value = response.data
                })
}

//search function
const searchCountry = computed(() => {
    let search = searchName.value.trim().toLowerCase()
    if(!search) return countries.value

    return searchObjectsByKey(countries.value, search)

})

//help function search
function searchObjectsByKey(array, searchTerm) {
    const results = [];
    for (const obj of array) {
        const result = searchObject(obj.name, searchTerm);
        if (result)  results.push(obj);
    }
    return results;
}
function searchObject(obj, searchTerm) {
    for (const key in obj) {
        if (typeof obj[key] === 'object') {
            const result = searchObject(obj[key], searchTerm);
            if (result) return result;
            
        } else if (typeof obj[key] === 'string' && obj[key].toLowerCase().includes(searchTerm.toLowerCase())) {
            return true;
        }
    }
    return false;
}

//watch for changing sortType that user select
watch(sortType, (newType, oldType) => {
    console.log('watch work')
    if(newType == 'asc'){
        countries.value = countries.value.sort(sortByNameAsc)
    }
    else if(newType == 'desc'){
        countries.value = countries.value.sort(sortByNameDesc)
    }
    console.log('old type',oldType)
})

//sort function
function sortByNameAsc(a, b) {
    return a.name.official.localeCompare(b.name.official);
}
function sortByNameDesc(a, b) {
    return b.name.official.localeCompare(a.name.official);
}

const totalPages = computed(() => {
      return Math.ceil(searchCountry.value.length / rowPerPage);
});

const paginatedData = computed(() => {
    const startIndex = (currentPage.value - 1) * rowPerPage;
    const endIndex = startIndex + rowPerPage;
    return searchCountry.value.slice(startIndex, endIndex);
})

const prePage = () => {
    if(currentPage.value > 1){
        currentPage.value --
    }
}

const nextPage = () => {
    if (currentPage.value < totalPages.value) {
        currentPage.value++;
      }
}

const showInfo = (country) => {
    object.value = country
    console.log('display object')
    console.log(object);
    document.getElementById("myModal").style.display = "block"

}

const closeModal = () => {
    document.getElementById("myModal").style.display = "none"
}


</script>
<style scoped>
.modal {
  display: none;
  position: fixed;
  z-index: 1;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgb(0,0,0);
  background-color: rgba(0,0,0,0.4);
}

.modal-content {
  background-color: #fefefe;
  margin: auto;
  padding: 20px;
  border: 1px solid #888;
  width: 70%;
  text-align: left;
}

.close {
  color: #aaaaaa;
  float: right;
  font-size: 28px;
  font-weight: bold;
}

.close:hover,
.close:focus {
  color: #000;
  text-decoration: none;
  cursor: pointer;
}

table{
    border: 1px solid gray;
    border-collapse: collapse;
    width: 100%;
}
th, td{
    border:1px solid gray;
    padding: 5px;
}
td{
    text-align: left;
}
.display-img{
    width: 60px;
}
.display-img img{
    width: 100%;
}
.search-input{
    outline: none;
    padding: 0.5rem;
    font-size: 18px;
}
.smalldata-container{
    display: flex;
    flex-wrap: wrap;
    gap: 2rem;
}
.header-search{
    display: flex;
    gap: 2rem;
    justify-content: center;
    margin-bottom: 2rem;
}
strong:first-letter{
    text-transform: uppercase;
}
.pagination-container{
    display: flex;
    gap: 1rem;
    justify-content: end;
    font-size: 20px;
    padding: 2rem;
}

</style>