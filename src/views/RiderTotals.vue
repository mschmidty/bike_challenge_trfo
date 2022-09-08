<template>
  <div>
    <section class="leader-section">
      <div class="most-activities">
        <h1>Most Activities</h1>
        <table>
          <thead>
            <tr>
              <th>Name</th>
              <th>Total Activities</th>
            </tr>
          </thead>
          <tbody v-for="person in sortedByActivity" :key="person.key">
            <tr>
              <td>{{person.key}}</td>
              <td>{{person.numActivities}}</td>
            </tr>
          </tbody>
        </table>
      </div>
      <div>
        <h1>Most Miles</h1>
        <table>
          <thead>
            <tr>
              <th>Name</th>
              <th>Total Miles</th>
            </tr>
          </thead>
          <tbody v-for="person in sortedByMiles" :key="person.key">
            <tr>
              <td>{{person.key}}</td>
              <td>{{person.totalMiles}}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>
    
    <hr>

    <h1>Individual Summaries</h1>
    <p>Way to get out there everyone! Keep it up.</p>
    <div v-for="person in riderData" :key="person.key" class="person-summary">
      <h3>{{person.key}}</h3>
      <p>Total Miles: {{person.totalMiles}} | Number of Activies: {{person.numActivities}}</p>
      <table>
        <thead>
          <tr>
            <th>Date</th>
            <th>Activity Type</th>
            <th>Distance (Miles)</th>
          </tr>
        </thead>
        
        <tbody v-for="activity in person.values" :key="activity['Completion Time']">
          <tr>
            <td>{{activity["Activity Date"]}}</td>
            <td>{{activity["Activity"]}}</td>
            <td>{{activity["Miles"]}}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
  import {csvParse} from "d3-dsv"
  import {tidy, groupBy, arrange, desc} from "@tidyjs/tidy"
  
  export default {
    data(){
      return {
        riderData:[],
        sortedByMiles:[],
        sortedByActivity:[]
      }
    },
    mounted(){
      fetch("/bike_challenge_data.csv")
        .then(response=>response.text())
        .then(text=>{
          const data = csvParse(text)
          const groupedData = tidy(
            data,
            groupBy('Name2', groupBy.entriesObject())
          )
          //console.log(groupedData)
          const finalData = groupedData.map(person=>{
            person["numActivities"]=person["values"].length
            //Total Miles
            const arrMiles = person["values"].map(activities=>{
              return parseFloat(activities.Miles)
            })
            person["totalMiles"] = arrMiles.reduce((a,b)=>{
              return a+b
            }, 0)
            return person
          })
          this.riderData=finalData

          this.sortedByMiles = tidy(
            finalData,
            arrange(desc('totalMiles'))
          )
          this.sortedByActivity=tidy(
            finalData,
            arrange([desc('numActivities')])
          )
        })
    }
   
  }
</script>

<style lang="scss" scoped>
  .person-summary{
    margin-top: 3rem;
  }
  .leader-section {
    display:flex;
    justify-content: space-between;
    flex-wrap:wrap;
  }
  .most-activities{
    margin-right: 2rem;
  }
</style>