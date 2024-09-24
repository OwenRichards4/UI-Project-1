<script>
import { Chart, registerables } from 'chart.js';
//import chartjs from 'chart.js@2.6.0';
let chartData;
import { onMount } from 'svelte';


let groupOption = "";
let groupName = "";
let pointsActivityCheck = true, goalsActivityCheck = true, assistsActivityCheck = true, savesActivityCheck = true, shotsActivityCheck = true, carActivityCheck = true, gamemodeActivityCheck = true; 
let activityName = "";
let backColor = "white"
let goalSelection = "", goalValue = 0;
let gamemodeSelection = "points";
let chartCanvas
let prevDate = ''
let xAxis = []
let displayedData = []
let dates = []
let scoreInput, savesInput, goalsInput, assistsInput, shotsInput;
let threeVthreeInput = false, twoVtwoInput = false, oneVoneInput = false;
let timer, clock;
let prevHeader, prevUl, newUl;
let headerVal, pointsVal, goalsVal, savesVal, shotsVal, assitsVal;
let datesArr = [], pointsArr = [], goalsArr = [], assistsArr = [], savesArr = [], shotsArr = [], carArr = [], gamemodeArr = [];
let date, points, goals, assists, saves, shots,  car2, gamemode2;
let errorMessage1 = "", errorMessage2 = "", errorMessage3 = "";
let pointsGoalValue = 0, goalsGoalValue = 0, assistsGoalValue = 0, savesGoalValue = 0, shotsGoalValue = 0;
let stats = ["Points", "Goals", "Assists", "Saves", "Shots", "Car", "Gamemode"];
let newInputs = new Map(), newArr = new Map(), newGoalCategory = new Map();
let carBorder;
let seconds = Number(localStorage.getItem("timer"))
let editCheck = false, id = -1;

let arr = [
  {
      date: '9/2/2024',
      points: 700,
      goals: 3,
      assists: 1,
      saves: 2,
      shots: 8,
      car: 'Fennec',
      group: 'Gameday',
      gamemode: '2v2'
  },
  {
      date: '9/3/2024',
      points: 500,
      goals: 2,
      assists: 1,
      saves: 1,
      shots: 7,
      car: 'Octane',
      group: 'Gameday',
      gamemode: '2v2'
  },
  {
      date: '9/4/2024',
      points: 300,
      goals: 0,
      assists: 1,
      saves: 1,
      shots: 3,
      car: 'Octane',
      group: 'Gameday',
      gamemode: '3v3'
  },
  {
      date: '9/6/2024',
      points: 600,
      goals: 2,
      assists: 1,
      saves: 2,
      shots: 6,
      car: 'Fennec',
      group: 'Scrim',
      gamemode: '1v1'
  },
  {
      date: '9/6/2024',
      points: 400,
      goals: 1,
      assists: 2,
      saves: 1,
      shots: 4,
      car: 'Octane',
      group: 'Scrim',
      gamemode: '3v3'
  }
];

    let car = "";
    let group = "";
    let dummyData = [];
    let groupArr = [];
    let theme = "dark";

    function updateClock() {
      let hoursTimer = 0; 
      let minutesTimer = 0; 
      let secondsTimer = 0;
      let hoursTimerString = ""; 
      let minutesTimerString = ""; 
      let secondsTimerString = "";

      let now = new Date();
      let hoursClock = now.getHours().toString().padStart(2, '0');
      let minutesClock = now.getMinutes().toString().padStart(2, '0');
      let secondsClock = now.getSeconds().toString().padStart(2, '0');

      const timeString = `${hoursClock}:${minutesClock}:${secondsClock}`;

      seconds = seconds + 1;

      localStorage.setItem("timer", seconds.toString())

      if (seconds >= 3600) {
        hoursTimer = Math.floor(seconds / 3600);
        let newMins = seconds - (hoursTimer * 3600);

        minutesTimer = Math.floor(newMins / 60);
        secondsTimer = newMins - (minutesTimer * 60);
            
        
        if (hoursTimer < 10) {
          hoursTimerString = "0" + hoursTimer;
        } else {
          hoursTimerString = hoursTimer.toString();
        }

        if (minutesTimer < 10) {
          minutesTimerString = "0" + minutesTimer
        } else {
          minutesTimerString = minutesTimer.toString()
        }
        
        if (secondsTimer < 10) {
          secondsTimerString = "0" + secondsTimer
        } else {
          secondsTimerString = secondsTimer.toString()
        }
      } else if (seconds >= 60) {
        hoursTimerString = "00"
        minutesTimer = Math.floor(seconds / 60)
        secondsTimer = seconds - (minutesTimer * 60)

        if (minutesTimer < 10) {
          minutesTimerString = "0" + minutesTimer
        } else {
          minutesTimerString = minutesTimer.toString()
        }
        
        if (secondsTimer < 10) {
          secondsTimerString = "0" + secondsTimer
        } else {
          secondsTimerString = secondsTimer.toString()
        }
      } else if (seconds < 60) {
        hoursTimerString = "00"
        minutesTimerString = "00"
        secondsTimer = seconds;

        if (secondsTimer < 10) {
          secondsTimerString = "0" + secondsTimer
        } else {
          secondsTimerString = secondsTimer.toString()
        }
      }
      let day = now.getDate().toString()
      let year = now.getFullYear()
      let month = now.getMonth()

      let date = month + "/" + day + "/" + year;

      let time = hoursTimerString + ":" + minutesTimerString + ":" + secondsTimerString

      timer = time;

      clock = date + "\xa0\xa0\xa0" + timeString;
  }
  setInterval(updateClock, 1000);

  // ################################## RESET LOCAL STORAGE ################################################
  
  //localStorage.removeItem('');
  
  for (let i = 0 ; i < localStorage.length; i++) {
      //console.log(localStorage.key(i))
      //console.log(localStorage.getItem(localStorage.key(i)));
      try {
        let obj = JSON.parse(localStorage[i]);
        dummyData.push(obj);
      }
      catch {
          
      }
  }

  let currGroups = document.querySelectorAll("[id='groupOptions']")
  currGroups.forEach(element => {
    if (!groupArr.includes(groupOption)){
      groupArr.push(groupOption);
    }
    dummyData.forEach(element2 => {
      if (!groupArr.includes(element2.group)){
        groupArr.push(element2.group);
      }
    });

    localStorage.setItem('group', groupArr.toString())
  });

  function addOption() {
    groupArr.forEach(element => {
        if (element != "new" && element != "") {
            let newElement = document.createElement('option')
            newElement.text = newElement.value = newElement.innerHTML = element;
            newElement.id = "groupOptions";
            document.getElementById("group-input").appendChild(newElement);
        } 
    });
  }
  addOption();

  function resetStats() {
    datesArr = [], pointsArr = [], goalsArr = [], assistsArr = [], savesArr = [], shotsArr = [], carArr = [], gamemodeArr = [];

    let dummyLen = dummyData.length
    for (let i = dummyLen -1; i > -1; i--){
      datesArr.push(dummyData[i].date);
      pointsArr.push(dummyData[i].points);
      goalsArr.push(dummyData[i].goals);
      assistsArr.push(dummyData[i].assists);
      savesArr.push(dummyData[i].saves);
      shotsArr.push(dummyData[i].shots);
      carArr.push(dummyData[i].car);
      gamemodeArr.push(dummyData[i].gamemode);
    }
  }
  resetStats();

  Chart.register(...registerables);
  let chart;

  onMount(() => {
    chart = new Chart("myChart", {
      type: "line",
      data: {
          labels: xAxis,
          datasets: [{
              data: displayedData,
              borderColor: "rgb(200,40,50)",
              fill: false
          }]
      },
      options: {
        //legend: {display: false}
      }
    });
  });

  function changeStats(category) {
    // console.log(category+"Check")
    // if (activityCheck == false)
    //     document.getElementById(category).style.display = "none";
    // else {
    //     document.getElementById(category).style.display = "flex";
    // }
  }

  function addActivity() {
    document.getElementById('activitiesName').style.display = "block";
    document.getElementById('activitiesButton').style.display = "block";
  }
  
  function createActivity() {
      let activitiesName = activityName;

      let newAct = document.createElement("li");

      let button = document.createElement("input");
      button.type = "checkbox";
      button.id = activitiesName + "Check";
      button.className = "activitiesCheckbox";
      button.checked = true;
      
      button.addEventListener('click', function() {
        changeStats(activitiesName);
      })

      let label = document.createElement("label");
      label.htmlFor = button.id;
      label.innerHTML = activitiesName;
      stats = [...stats, activitiesName]

      let group = document.createElement('div');
      group.className = "activitiesItem"

      group.appendChild(button);
      group.appendChild(label);

      //document.getElementById(button.id).onclick = changeStats(activitiesName)

      document.getElementById('activitesList').appendChild(group)

      document.getElementById('activitiesName').style.display = "none";
      document.getElementById('activitiesButton').style.display = "none";

      let item = document.getElementById('lastItem');

      document.getElementById('activitesList').insertBefore(group, item)
      
      let newStat = document.createElement('div');
      newStat.id = activitiesName;
      newStat.className = "data-stats";

      let header = document.createElement('h3');
      header.innerHTML = activitiesName;
      newStat.appendChild(header);

      for (let i = dummyData.length - 1; i > -1 ; i--) {
        let newH3 = document.createElement('h3')
        newH3.className = i.toString();
        newH3.id = activitiesName;
        newH3.innerHTML = "-";

        newStat.appendChild(newH3);
      }
      //console.log(newStat)

      newArr.set(activitiesName+"array", newStat)
      document.getElementById('stats').insertBefore(newStat, document.getElementById('stats').lastElementChild);

      let div = document.createElement('div');
      div.id = div.className = 'input-items'

      const newInput = document.createElement('input');
      newInput.id = activitiesName + "input"
      newInputs.set(activitiesName + "input", newInput)

      let label2 = document.createElement('label');
      label2.id = activitiesName + "-id"
      label2.innerHTML = activitiesName
      label2.htmlFor = newInput.id

      div.appendChild(label2);
      div.appendChild(newInput);
      document.getElementById('game-stats').appendChild(div);

      document.getElementById(activitiesName + "-id").style.fontSize = "150%";

      document.getElementById('game-stats').style.display = "flex";
      document.getElementById('game-stats').style.alignItems = "center";

      for (let i = 0; i < dummyData.length; i++) {
        let test = localStorage.getItem(i.toString());
        let test2 = {}
        test2 = JSON.parse(test);
        //console.log(i);
        test2.activitiesName = " ";
        test = JSON.stringify(test2)
        localStorage.setItem(i.toString(), test)
      }

      let newOption = document.createElement('option');
      newOption.innerHTML = activitiesName;
      newOption.value = activityName;
      document.getElementById('gamemode2').appendChild(newOption);
      document.getElementById('goalSelection').appendChild(newOption);

      activityName = "";
  }

  function themeChange() {
      if (theme == "dark") {
          theme = "light";
          document.getElementById('graph-box').style.backgroundColor = "rgb(238, 238, 238)"
          document.getElementById('add-game-box').style.backgroundColor = "rgb(238, 238, 238)"
          document.getElementById('past-stats').style.backgroundColor = "rgb(238, 238, 238)"
          document.getElementById('pie-chart').style.backgroundColor = "rgb(238, 238, 238)"
          document.getElementById('activities').style.backgroundColor = "rgb(238, 238, 238)"

          document.getElementById('graph-box').style.borderColor = "rgb(200,40,50)"
          document.getElementById('add-game-box').style.borderColor = "rgb(200,40,50)"
          document.getElementById('past-stats').style.borderColor = "rgb(200,40,50)"
          document.getElementById('pie-chart').style.borderColor = "rgb(200,40,50)"
          document.getElementById('activities').style.borderColor = "rgb(200,40,50)"

          document.getElementById('graph-box').style.boxShadow = "rgb(200,40,50) 0px 3px 3px"
          document.getElementById('add-game-box').style.boxShadow = "rgb(200,40,50) 0px 3px 3px"
          document.getElementById('past-stats').style.boxShadow = "0px 3px 3px rgb(200,40,50)"
          document.getElementById('pie-chart').style.boxShadow = "0px 3px 3px rgb(200,40,50)"
          document.getElementById('activities').style.boxShadow = "0px 3px 3px rgb(200,40,50)"

          backColor = "rgb(22, 22, 22)";

          // var elms = document.querySelectorAll("[id='headerEle']");
          // for(var i = 0; i < elms.length; i++) {
          //   backColor = "rgb(22, 22, 22)";
          // }
          // var elms = document.querySelectorAll("[id='labels']");
          // for(var i = 0; i < elms.length; i++) {
          //   backColor = "rgb(22, 22, 22)";
          // }
          // var elms = document.querySelectorAll("[id='lines']");
          // for(var i = 0; i < elms.length; i++) {
          //   backColor = "rgb(22, 22, 22)";
          // }
          // var elms = document.querySelectorAll("[id='actLabel']");
          // for(var i = 0; i < elms.length; i++) {
          //   backColor = "rgb(22, 22, 22)";
          // }

          document.getElementById('lastItem').style.color = "rgb(22, 22, 22)"
          document.getElementById('stats').style.color = "rgb(22, 22, 22)"
              
      } else {
          theme = "dark";
          document.getElementById('graph-box').style.backgroundColor = "rgb(22, 22, 22)"
          document.getElementById('add-game-box').style.backgroundColor = "rgb(22, 22, 22)"
          document.getElementById('past-stats').style.backgroundColor = "rgb(22, 22, 22)"
          document.getElementById('pie-chart').style.backgroundColor = "rgb(22, 22, 22)"
          document.getElementById('activities').style.backgroundColor = "rgb(22, 22, 22)"

          document.getElementById('graph-box').style.borderColor = "white"
          document.getElementById('add-game-box').style.borderColor = "white"
          document.getElementById('past-stats').style.borderColor = "white"
          document.getElementById('pie-chart').style.borderColor = "white"
          document.getElementById('activities').style.borderColor = "white"

          document.getElementById('graph-box').style.boxShadow = "white 0px 3px 3px"
          document.getElementById('add-game-box').style.boxShadow = "white 0px 3px 3px"
          document.getElementById('past-stats').style.boxShadow = "0px 3px 3px white"
          document.getElementById('pie-chart').style.boxShadow = "0px 3px 3px white"
          document.getElementById('activities').style.boxShadow = "0px 3px 3px white"

          backColor = "white";

          // var elms = document.querySelectorAll("[id='headerEle']");
          // for(var i = 0; i < elms.length; i++) {
          //   backColor = "white";
          // }
          // var elms = document.querySelectorAll("[id='labels']");
          // for(var i = 0; i < elms.length; i++) {
          //   backColor = "white";
          // }
          // var elms = document.querySelectorAll("[id='lines']");
          // for(var i = 0; i < elms.length; i++) {
          //   backColor = "white";
          // }
          // var elms = document.querySelectorAll("[id='actLabel']");
          // for(var i = 0; i < elms.length; i++) {
          //   backColor = "white";
          // }

          document.getElementById('stats').style.color = "white"
          document.getElementById('lastItem').style.color = "white"
          
      }
  }

  //localStorage.removeItem("41");

  function updateData(obj) {
    dummyData.push(obj);
    let cap = 0;
    let check = false;
    for (let i = 0 ; i < localStorage.length; i++) {
      if (localStorage.getItem(i.toString())){
        if(i > cap) {
          cap = i;
        }
      }
    }
    localStorage.setItem((cap + 1).toString(), JSON.stringify(obj));
  }

  // function updateStats() {
  //   datesArr = [], pointsArr = [], goalsArr = [], assistsArr = [], savesArr = [], shotsArr = [], carArr = [], gamemodeArr  = [];
  //   for (let i = dummyData.length - 1; i > 0; i--) {
  //     //console.log(localStorage.getItem(i))
  //     //document.getElementById("date-" + i).innerHTML = localStorage.getItem(i);

  //     datesArr.push(dummyData[dummyData.length - i].date);
  //     pointsArr.push(dummyData[dummyData.length - i].points);
  //     goalsArr.push(dummyData[dummyData.length - i].goals);
  //     assistsArr.push(dummyData[dummyData.length - i].assists);
  //     savesArr.push(dummyData[dummyData.length - i].saves);
  //     shotsArr.push(dummyData[dummyData.length - i].shots);
  //     carArr.push(dummyData[dummyData.length - i].car);
  //     gamemodeArr.push(dummyData[dummyData.length - i].gamemode);
  //   }
  // }

  function graphChange() {
    let sum = 0;
    let count = 0;
    displayedData = []
    xAxis = []

    dummyData.forEach(element => {
      if (prevDate != element.date) {
        if (count > 0) {
          displayedData[displayedData.length - 1] = sum / count;
        }
        xAxis.push(element.date);

        if (gamemodeSelection == "points") {
          displayedData.push(element.points);
          sum = element.points;
        } else if (gamemodeSelection == "goals") {
          displayedData.push(element.goals);
          sum = element.goals;
        } else if (gamemodeSelection == "assists") {
          displayedData.push(element.assists);
          sum = element.assists;
        } else if (gamemodeSelection == "saves") {
          displayedData.push(element.saves);
          sum = element.saves;
        } else if (gamemodeSelection == "shots") {
          displayedData.push(element.shots);
          sum = element.shots;
        }
        count = 1;
      } else {
        count++;
        if (gamemodeSelection == "points") {
          sum += element.points;
        } else if (gamemodeSelection == "goals") {
          sum += element.goals;
        } else if (gamemodeSelection == "assists") {
          sum += element.assists;
        } else if (gamemodeSelection == "saves") {
          sum += element.saves;
        } else if (gamemodeSelection == "shots") {
          sum += element.shots;
        }
      }
      prevDate = element.date;
    });

  if (count > 1) {
    displayedData[displayedData.length - 1] = sum / count;
  }
  
  if (chart) {
    chart.data.labels = xAxis;
    chart.data.datasets[0].data = displayedData;
    chart.update();
  }

  }
  graphChange();

  $: if (gamemodeSelection) {
    graphChange();
  }

  function addStats() {
    document.getElementById('top-group').style.filter = "blur(10px)";
    document.getElementById('lower-group').style.filter = "blur(10px)";
    document.getElementById('header').style.filter = "blur(10px)";
    document.getElementById('footer').style.filter = "blur(10px)";
    let groupName = groupOption;
    if (groupName == "new") {
      if (groupOption) {
        // name was added to text box
        let newElement = document.createElement('option')
        newElement.text = newElement.value = groupName;
        newElement.id = "groupOptions";
        document.getElementById("group-input").appendChild(newElement);
        group = groupName;          
        document.getElementById('error2').style.visibility = "hidden"; 

        localStorage.setItem("groups", group)

        document.getElementById('group-input-text').style.visibility = "hidden";
        document.getElementById("popup").style.visibility =  "visible";
      } else {
        // name was NOT added to text box
        document.getElementById('error2').style.visibility = "visible";
        document.getElementById('lower-group').style.filter = "blur(0px)";
        document.getElementById('top-group').style.filter = "blur(0px)";
        document.getElementById('header').style.filter = "blur(0px)";
        document.getElementById('footer').style.filter = "blur(0px)";
        return;
      }
    } else {
      group = groupName;
      document.getElementById('popup').style.visibility =  "visible";
    }
  }

  function carSelected(carName) {
      if (carName == 'octane') {
          document.getElementById('fennecPic').style.border = "2px solid white";
          document.getElementById('dominusPic').style.border = "2px solid white";
          document.getElementById('other').style.border = "2px solid white";
          document.getElementById('fennec-text').style.color = "white";
          document.getElementById('dominus-text').style.color = "white";
          document.getElementById('other-text').style.color = "white";

          document.getElementById('octanePic').style.border = "2px solid rgb(200,40,50)";
          document.getElementById('octane-text').style.color = "rgb(200,40,50)";

          car = 'Octane';  
      } else if(carName == 'fennec') {
          document.getElementById('octanePic').style.border = "2px solid white";
          document.getElementById('dominusPic').style.border = "2px solid white";
          document.getElementById('other').style.border = "2px solid white";
          document.getElementById('octane-text').style.color = "white";
          document.getElementById('dominus-text').style.color = "white";
          document.getElementById('other-text').style.color = "white";

          document.getElementById('fennecPic').style.border = "2px solid rgb(200,40,50)";
          document.getElementById('fennec-text').style.color = "rgb(200,40,50)";

          car = 'Fennec';
      } else if (carName == 'dominus') {
          document.getElementById('fennecPic').style.border = "2px solid white";
          document.getElementById('octanePic').style.border = "2px solid white";
          document.getElementById('other').style.border = "2px solid white";
          document.getElementById('fennec-text').style.color = "white";
          document.getElementById('octane-text').style.color = "white";
          document.getElementById('other-text').style.color = "white";

          document.getElementById('dominusPic').style.border = "2px solid rgb(200,40,50)";
          document.getElementById('dominus-text').style.color = "rgb(200,40,50)";

          car = 'Dominus';
      } else if (carName == 'other') {
          document.getElementById('fennecPic').style.border = "2px solid white";
          document.getElementById('octanePic').style.border = "2px solid white";
          document.getElementById('dominusPic').style.border = "2px solid white";
          document.getElementById('fennec-text').style.color = "white";
          document.getElementById('octane-text').style.color = "white";
          document.getElementById('dominus-text').style.color = "white";

          document.getElementById('other').style.border = "2px solid rgb(200,40,50)";
          document.getElementById('other-text').style.color = "rgb(200,40,50)";

          car = 'Other';
      }
  }

  function submit() {
    let count = 0;
    let gamemode;
    groupName = "";

    if (!scoreInput && scoreInput != 0) {
      document.getElementById('error').style.visibility = "visible";
      errorMessage1 = "Missing Input: Score";
      count++;
    }
    if (!goalsInput && goalsInput != 0) {
      document.getElementById('error').style.visibility = "visible";
      errorMessage1 = "Missing Input: Goals";
      count++;
    } 
    if (!assistsInput && assistsInput != 0) {
      document.getElementById('error').style.visibility = "visible";
      errorMessage1 = "Missing Input: Assists";
      count++;
    } if (!savesInput && savesInput != 0) {
      document.getElementById('error').style.visibility = "visible";
      errorMessage1 = "Missing Input: Saves";
      count++;
    } 
    if (!shotsInput && shotsInput != 0) {
      document.getElementById('error').style.visibility = "visible";
      errorMessage1 = "Missing Input: Shots";
      count++;
    }
    if (count > 1) {
      document.getElementById('error').style.visibility = "visible";
      errorMessage1 = "Multiple Missing Inputs";
    }

    let temp = {};
    
    if (count == 0) {
      if (oneVoneInput && !twoVtwoInput && !threeVthreeInput) {
        gamemode = "1v1";
      } else if (!oneVoneInput && twoVtwoInput && !threeVthreeInput) {
        gamemode = "2v2";
      } else if (!oneVoneInput && !twoVtwoInput && threeVthreeInput) {
        gamemode = "3v3";
      } else if (!oneVoneInput && !twoVtwoInput && !threeVthreeInput) {
        if (car) {
          document.getElementById('error').style.visibility = "visible";
          errorMessage1 = "Missing Input: Gamemode";
        } else {
          document.getElementById('error').style.visibility = "visible";
          errorMessage1 = "Missing Multiple Inputs";
        }
      } else {
        document.getElementById('error').style.visibility = "visible";
        errorMessage1 = "Multiple Gamemodes Selected";
      }
      if (gamemode) {
        if (car) {

          let currentdate = new Date(); 
          let currDate = (currentdate.getMonth()+1)  + "/" 
            + currentdate.getDate() + "/"
            + currentdate.getFullYear();

          for (let i = 0 ; i < localStorage.length; i++) {
            // console.log(localStorage.key(i))
            // console.log(localStorage.getItem(localStorage.key(i)));
          }

          if (editCheck == false) {

            stats.forEach(element => {
              if (element != "Points" && element != "Goals" && element != "Assists" && element != "Saves" && element != "Shots" && element != "Car" && element != "Gamemode" ){
                let val = newInputs.get(element.toString() + "input").value
                temp[element] = val;

                let x = document.createElement('h3')
                x.innerHTML = val;

                newArr.get(element+"array").insertBefore(x, newArr.get(element+"array").children[1]);
                
                let newStat = newArr.get(element +"array");
              }
            });

            temp = {
              date: currDate,
              points: scoreInput,
              goals: goalsInput,
              assists: assistsInput,
              saves: savesInput,
              shots: shotsInput,
              car: car,
              group: groupName,
              gamemode: gamemode
            }

            datesArr = datesArr.reverse();
            pointsArr = pointsArr.reverse();
            goalsArr = goalsArr.reverse();
            assistsArr = assistsArr.reverse();
            savesArr = savesArr.reverse();
            shotsArr = shotsArr.reverse();
            carArr = carArr.reverse();
            gamemodeArr = gamemodeArr.reverse();


            datesArr = [...datesArr, currDate].reverse();
            pointsArr = [...pointsArr, scoreInput].reverse();
            goalsArr = [... goalsArr, goalsInput].reverse();
            assistsArr = [...assistsArr, goalsInput].reverse();
            savesArr = [...savesArr, savesInput].reverse();
            shotsArr = [...shotsArr, shotsInput].reverse();
            carArr = [...carArr, car].reverse();
            gamemodeArr = [...gamemodeArr, gamemode].reverse();

            updateData(temp);
          } else {

            stats.forEach(element => {
              if (element != "Points" && element != "Goals" && element != "Assists" && element != "Saves" && element != "Shots" && element != "Car" && element != "Gamemode" ){
                let val = newInputs.get(element.toString() + "input").value
                temp[element] = val;

                let x = document.createElement('h3')
                x.innerHTML = val;

                let idName = "";
                idName = element.toString() + (dummyData.length - id - 1).toString();

                let newString = element.toString()
                //console.log(idName, val);
                let y = document.querySelectorAll("[id="+element+"]")
                
                

                y.forEach(child => {
                  if (child.className == (dummyData.length - id - 1).toString()) {
                    child.innerHTML = val;
                  }
                });
                // newArr.get(element+"array").forEach(child => {
                //   console.log(child);
                // })
              }
            });

            dummyData[dummyData.length - id - 1].points = scoreInput;
            dummyData[dummyData.length - id - 1].goals = goalsInput;
            dummyData[dummyData.length - id - 1].assists = assistsInput;
            dummyData[dummyData.length - id - 1].saves = savesInput;
            dummyData[dummyData.length - id - 1].shots = shotsInput;
            dummyData[dummyData.length - id - 1].car = car;
            dummyData[dummyData.length - id - 1].gamemode = gamemode;

            let obj = JSON.parse(localStorage.getItem((dummyData.length - id - 1).toString()));

            
  
            obj.points = scoreInput;
            obj.goals = goalsInput;
            obj.assists = assistsInput;
            obj.saves = savesInput;
            obj.shots = shotsInput;
            obj.car = car;
            obj.gamemode = gamemode;
            

            stats.forEach(element => {
              if (element != "Points" && element != "Goals" && element != "Assists" && element != "Saves" && element != "Shots" && element != "Car" && element != "Gamemode") {
                let val = newInputs.get(element.toString() + "input").value
                obj.element = val; 
              }
            });

            resetStats();
            editCheck = false;

            obj = JSON.stringify(obj)
            localStorage.setItem((dummyData.length - id - 1).toString(), obj)

            //stats.removeChild(stats.lastElementChild);
          }

          scoreInput = goalsInput = assistsInput = savesInput = shotsInput = "";

          oneVoneInput = twoVtwoInput = threeVthreeInput = false;

          carBorder = "2px solid white";

          document.getElementById('octane-text').style.color = "white";
          document.getElementById('fennec-text').style.color = "white";
          document.getElementById('dominus-text').style.color = "white";
          document.getElementById('other-text').style.color = "white";

          document.getElementById('error').style.visibility = "hidden";
          document.getElementById('popup').style.visibility = "hidden";
          document.getElementById('lower-group').style.filter = "blur(0px)";
          document.getElementById('top-group').style.filter = "blur(0px)";
          document.getElementById('header').style.filter = "blur(0px)";
          document.getElementById('footer').style.filter = "blur(0px)";
          graphChange();

          
          //updateStats();
        } else {
          document.getElementById('error').style.visibility = "visible";
          document.getElementById('error').innerHTML = "Missing Input: Car";
        }
      }
    }
    car = "";
  }

  function cancel() {
    document.getElementById('popup').style.visibility = "hidden";
    document.getElementById('error').style.visibility = "hidden";
    document.getElementById('lower-group').style.filter = "blur(0px)";
    document.getElementById('top-group').style.filter = "blur(0px)";
    document.getElementById('header').style.filter = "blur(0px)";
    document.getElementById('footer').style.filter = "blur(0px)";
  }

  function historyPage() {
    //window.location = "history.html";
  }

  function editHistory(event) {
    id = event.target.id;
    editCheck = true;
    
    scoreInput = dummyData[dummyData.length - id - 1].points
    goalsInput = dummyData[dummyData.length - id - 1].goals
    assistsInput = dummyData[dummyData.length - id - 1].assists
    savesInput = dummyData[dummyData.length - id - 1].saves
    shotsInput = dummyData[dummyData.length - id - 1].shots

    carSelected((dummyData[dummyData.length - id - 1].car).toLowerCase())
    
    let mode = dummyData[dummyData.length - id - 1].gamemode
    if (mode == '1v1') {
      oneVoneInput = true;
    } else if (mode == '2v2') {
      twoVtwoInput = true;
    } else {
      threeVthreeInput = true;
    }

    document.getElementById('top-group').style.filter = "blur(10px)";
    document.getElementById('lower-group').style.filter = "blur(10px)";
    document.getElementById('header').style.filter = "blur(10px)";
    document.getElementById('footer').style.filter = "blur(10px)";
    let groupName = groupOption;
    if (groupName == "new") {
      if (groupOption) {
        // name was added to text box
        let newElement = document.createElement('option')
        newElement.text = newElement.value = groupName;
        newElement.id = "groupOptions";
        document.getElementById("group-input").appendChild(newElement);
        group = groupName;          
        document.getElementById('error2').style.visibility = "hidden"; 

        localStorage.setItem("groups", group)

        document.getElementById('group-input-text').style.visibility = "hidden";
        document.getElementById("popup").style.visibility =  "visible";
      } else {
        // name was NOT added to text box
        document.getElementById('error2').style.visibility = "visible";
        document.getElementById('lower-group').style.filter = "blur(0px)";
        document.getElementById('top-group').style.filter = "blur(0px)";
        document.getElementById('header').style.filter = "blur(0px)";
        document.getElementById('footer').style.filter = "blur(0px)";
        return;
      }
    } else {
      group = groupName;
      document.getElementById('popup').style.visibility =  "visible";
    }
  }

  for (let i = 0; i < arr.length; i++) {
    localStorage.setItem(i.toString(), JSON.stringify(arr[i]))
  }

  function createGoal(type) {
    if (type == "points") {
      pointsGoalValue = goalValue;
    }
    else if (type == "goals") {
      goalsGoalValue = goalValue;
    }
    else if (type == "assists") {
      assistsGoalValue = goalValue;
    }
    else if (type == "saves") {
      savesGoalValue = goalValue;
    }
    else if (type == "shots") {
      shotsGoalValue = goalValue;
    } else {
      //console.log(document.getElementById('goalSelection').value);
      let val = document.getElementById('goalSelection').value;
      let y = document.querySelectorAll("[id="+val+"]")
      
      y.forEach(child => {
        
        if (child.innerHTML != "-" ) { 
          if (Number(child.innerHTML) > Number(goalValue)) {
            child.style.color = "green";
          } else if (Number(child.innerHTML) < Number(goalValue)) {
            child.style.color = "red";
          } else {
            child.style.color = "yellow";
          }
        }
      });
      // if (document.getElementById(type)) {
      //   document.getElementById(type).forEach(element => {
          
      //   });
      // }
    }
  }

</script>

<header id='header'>
  <h1 style="position: relative; min-width: 60%;">Rocket League Statistics Tracker</h1>
  <nav>
    <ul style="list-style: none;">
      <div class="list-item">
      <!--  <ion-icon name="bar-chart-outline"></ion-icon>
        <li>Dashboard</li>
      </div>
      <li>|</li>
      <div class="list-item">
        <ion-icon name="file-tray-full-outline" onclick="history()"></ion-icon>
        <li on:click={historyPage}>History</li>
      </div>
      <li>|</li>-->
      <div class="list-item">
        <button on:click={themeChange}>Theme Toggle</button>
      </div>
    </ul>
  </nav>
</header>
<svg class="original-line" width="95%" height="2px" style="position: relative; left: 2.5%; bottom: 48px;">
  <rect width="100%" height="100%" fill="white" ></rect>
</svg>
<main>
  <div class="top-group" id="top-group">
      <div class="graph-box" id="graph-box">
        <div style="display: flex; flex-direction: row; align-items: center; justify-content: end; gap: 30px;">
          <div>
            <label id="labels" for="group-input" style:backgroundColor={backColor} style:color={backColor} style="z-index: 2;">Group</label>
            <select name="group" class="group" id="group-input" bind:value={groupOption}> 
              <option id="groupOptions" value=""></option>
              <option id="groupOptions" value='new'>New</option>
            </select>
          </div>
          <div>
            <div style="display: flex; flex-direction: row; justify-content: end; margin-right: 20px; z-index: 2;">
              <label id="labels" for="gamemode" style:backgroundColor={backColor} style:color={backColor}>Category</label>
              <select on:change={graphChange} bind:value={gamemodeSelection} id="gamemode2" name="gamemode" class="gamemode">
                <option value="points">Points</option>
                <option value="goals">Goals</option>
                <option value="assists">Assists</option>
                <option value="saves">Saves</option>
                <option value="shots">Shots</option>
              </select>
            </div>
          </div>
        </div>
        <br><br>
        <canvas class="chart" id="myChart" bind:this={chartCanvas} style=" height:100%; width:100%;"></canvas>
      </div>
      <div class="add-game-box" id="add-game-box">
        <h2 id="headerEle" style="font-size: 180%; " style:color={backColor}>Add a new game</h2>
        
        <div id="group-stuff" style="padding: 0 0 15px 0; display: flex; flex-direction: row; width:fit-content; justify-content: space-evenly;">
          <label id="labels" style="font-size: 140%;" for="group-input" style:backgroundColor={backColor} style:color={backColor}>Group</label>
          <select style="font-size: 100%; margin-left: 5px; width: 30%;" name="group" class="group" id="group-input" bind:value={groupOption}> 
            <option id="groupOptions" value=""></option>
            <option id="groupOptions" value="new">New</option>
          </select>
          {#if groupOption == "new"}
            <input type="text" style="position: relative; margin-left: 5px; visibility: visible; display: block;" bind:value={groupName} id="group-input-text" placeholder="Name">
          {:else}
            <input type="text" style="position: relative; margin-left: 5px; visibility: hidden; display: none;" bind:value={groupName} id="group-input-text" placeholder="Name">
          {/if}
          
        </div>
        <div style="position: relative; width: 100%; display: flex; flex-direction: column; justify-content: start; align-items: start; bottom:10px;">
            <button style="position: relative;" class="addstats" on:click={addStats}>Add new stats</button>
            <div class="error2" id="error2" style="position: relative; text-align: center; background-color: red; padding: 2px 0; visibility: hidden; width: 60%; left: 20%; margin-top: 10px;"> New Group Name Needed</div>
        </div>
      </div>
  </div>
  <div class="lower-group" id="lower-group">
    <div class="past-stats" id="past-stats">
      <h2 id="headerEle" style="padding-left: 20px;" style:color={backColor}>Past Statistics</h2>
        <!--<div class="stats-header">
          {#each stats as stat}<h3>{stat}</h3>{/each}
        </div>-->
        <div id='stats' class="stats">
          <div class="data-stats" id="date" style="position: relative; top: 58px;">
            {#each datesArr as date, i}
                <h3>{date}</h3> 
            {/each}
          </div>
          {#if pointsActivityCheck == true}
            <div class="data-stats" id="points">
              <h3>Points</h3>
              {#each pointsArr as points, i}
                {#if pointsGoalValue != 0}
                  {#if pointsGoalValue > points}
                    <h3 style:color="red">{points}</h3>
                  {:else if pointsGoalValue == points}
                    <h3 style:color="yellow">{points}</h3>
                  {:else}
                    <h3 style:color="green">{points}</h3>
                  {/if}
                {:else}
                  <h3>{points}</h3>
                {/if}
              {/each}
            </div>
          {/if}
          {#if goalsActivityCheck == true}
            <div class="data-stats" id="goals">
              <h3>Goals</h3>
              {#each goalsArr as goals, i}
                {#if goalsGoalValue != 0}
                  {#if goalsGoalValue > goals}
                    <h3 style:color="red" id={'goals-' + i}>{goals}</h3>
                  {:else if goalsGoalValue == goals}
                    <h3 style:color="yellow" id={'goals-' + i}>{goals}</h3>
                  {:else}
                    <h3 style:color="green" id={'goals-' + i}>{goals}</h3>
                  {/if}
                {:else}
                  <h3>{goals}</h3>
                {/if}
              {/each}
            </div>
          {/if}
          {#if assistsActivityCheck == true}
            <div class="data-stats" id="assists">
              <h3>Assists</h3>
              {#each assistsArr as assists, i}
                {#if assistsGoalValue != 0}
                  {#if assistsGoalValue > assists}
                    <h3 style:color="red">{assists}</h3>
                  {:else if assistsGoalValue == assists}
                    <h3 style:color="yellow">{assists}</h3>
                  {:else}
                    <h3 style:color="green">{assists}</h3>
                  {/if}
                {:else}
                  <h3>{assists}</h3>
                {/if}
              {/each}
            </div>
          {/if}
          {#if savesActivityCheck == true}
            <div class="data-stats" id="saves">
              <h3>Saves</h3>
              {#each savesArr as saves, i}
                {#if savesGoalValue != 0}
                  {#if savesGoalValue > saves}
                    <h3 style:color="red">{saves}</h3>
                  {:else if savesGoalValue == saves}
                    <h3 style:color="yellow">{saves}</h3>
                  {:else}
                    <h3 style:color="green">{saves}</h3>
                  {/if}
                {:else}
                  <h3>{saves}</h3>
                {/if}
              {/each}
            </div>
          {/if}
          {#if shotsActivityCheck == true}
            <div class="data-stats" id="shots">
              <h3>Shots</h3>
              {#each shotsArr as shots, i}
                {#if shotsGoalValue != 0}
                  {#if shotsGoalValue > shots}
                    <h3 style:color="red">{shots}</h3>
                  {:else if shotsGoalValue == shots}
                    <h3 style:color="yellow">{shots}</h3>
                  {:else}
                    <h3 style:color="green">{shots}</h3>
                  {/if}
                {:else}
                  <h3>{shots}</h3>
                {/if}
              {/each}
            </div>
          {/if}
          {#if carActivityCheck == true}
            <div class="data-stats" id="car">
              <h3>Car</h3>
              {#each carArr as car2, i}
                <h3>{car2}</h3>
              {/each}
            </div>
          {/if}
          {#if gamemodeActivityCheck}
            <div class="data-stats" id="gamemode">
              <h3>Gamemode</h3>
              {#each gamemodeArr as gamemode2, i}
                <h3>{gamemode2}</h3>
              {/each}
            </div>
          {/if}
            <div class="data-stats" id="gamemode">
            {#each gamemodeArr as gamemode2, i}
              <h3 class='edit' id={i.toString()} on:click={editHistory}>Edit</h3>
            {/each}
          </div>
        </div>
        <!--
        <svg class="stats-underline-1" id="lines" style:fill={backColor} viewBox="0 0 100 0.01">
            <rect width="110" height="2"></rect>
        </svg>
        <svg class="stats-underline-2" id="lines" style:fill={backColor} viewBox="0 0 100 0.01">
            <rect width="110" height="2"></rect>
        </svg>
        <svg class="stats-underline-3" id="lines" style:fill={backColor} viewBox="0 0 100 0.01">
            <rect width="110" height="2"></rect>
        </svg>
        <svg class="stats-underline-4" id="lines" style:fill={backColor} viewBox="0 0 100 0.01">
            <rect width="110" height="2"></rect>
        </svg>
        <svg class="stats-underline-5" id="lines" style:fill={backColor} viewBox="0 0 100 0.01">
            <rect width="110" height="2"></rect>
        </svg>
        -->
    </div>
    <div style="position: relative; display: flex; flex-direction: column; justify-content: space-between; align-items: stretch; gap: 30px; width: 30%;">
      <div class="pie-chart" id="pie-chart" style="overflow: hidden; width: 100%;">
        <h2 id="headerEle" style="text-align: center;" style:color={backColor}>Goals</h2>
        
        <div style="display: flex; flex-direction: row; justify-content: center; gap: 20px;">
          <label id="labels" for="goalSelection" style:backgroundColor={backColor} style:color={backColor}>Category</label>
          <select id="goalSelection" name="goalSelection" class="goalSelection" bind:value={goalSelection}>
            <option value="points">Points</option>
            <option value="goals">Goals</option>
            <option value="assists">Assists</option>
            <option value="saves">Saves</option>
            <option value="shots">Shots</option>
          </select>
          <input id="goalValue" placeholder="Value" bind:value={goalValue}>
        </div>

        <button type="button" class="goalsButton" on:click={() => createGoal(goalSelection)}>Set a Goal</button>
        <div class="error3" id="error3" style="position: relative; text-align: center; background-color: red; padding: 2px 0; visibility: hidden;">{errorMessage3}</div>
        <!--<canvas id="myChart2" style="position: relative; width: 0%; max-height: fit-content-20px; max-width: 90%; left: 5%; top: 1.5%;"></canvas> -->
      </div>
      <div class="activities" id="activities">
        <div style="display: flex; justify-content: center; align-items: center;">
          <h2 id="headerEle" style="text-align: center;" style:color={backColor}>Activities</h2>
          <input id="activitiesName" bind:value={activityName} placeholder="new activity name" style="position: relative; height: fit-content; display: none; left: 10px;">
          <button id="activitiesButton" style="position: relative; height: fit-content; display: none; left: 10px;" on:click={createActivity}>Add</button>
        </div>
        <div id="activitesList" style="position: relative; display: grid; grid-template-columns: auto auto auto; gap: 20px; width: 100%;">
          <div class="activitiesItem">
            <input class="activitiesCheckbox" id="pointsCheck" type="checkbox" bind:checked={pointsActivityCheck}>
            <label id="actLabel" for="pointsCheck" style:color={backColor}>Points</label>
          </div>
          <div class="activitiesItem">
            <input class="activitiesCheckbox" id="goalsCheck" type="checkbox" bind:checked={goalsActivityCheck}>
            <label id="actLabel" for="goalsCheck" style:color={backColor}>Goals</label>
          </div>
          <div class="activitiesItem">
            <input class="activitiesCheckbox" id="assistsCheck" type="checkbox" bind:checked={assistsActivityCheck}>
            <label id="actLabel" for="assistsCheck" style:color={backColor}>Assists</label>
          </div>
          <div class="activitiesItem">
            <input class="activitiesCheckbox" id="savesCheck" type="checkbox" bind:checked={savesActivityCheck}>
            <label id="actLabel" for="savesCheck" style:color={backColor}>Saves</label>
          </div>
          <div class="activitiesItem">
            <input class="activitiesCheckbox" id="shotsCheck" type="checkbox" bind:checked={shotsActivityCheck}>
            <label id="actLabel" for="shotsCheck" style:color={backColor}>Shots</label>
          </div>
          <div class="activitiesItem">
            <input class="activitiesCheckbox" id="carCheck" type="checkbox" bind:checked={carActivityCheck}>
            <label id="actLabel" for="carCheck" style:color={backColor}>Car</label>
          </div>
          <div class="activitiesItem">
            <input class="activitiesCheckbox" id="gamemodeCheck" type="checkbox" bind:checked={gamemodeActivityCheck}>
            <label id="actLabel" for="gamemodeCheck" style:color={backColor}>Gamemode</label>
          </div>
          <div id="lastItem" class="activitiesItem">
            <label id="actLabel" on:click={() => addActivity()} style="cursor: pointer;" style:backgroundColor={backColor}>Add Activity</label>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div class="popup" id="popup">
    <h1 style="text-align: center; font-size: 300%; padding: 0;">New Game Stats</h1>
    <div class="game-stats" id="game-stats">
        <div class="input-items">
            <label for="score-input" style="font-size: 150%;">Score</label>
            <input type="number" id="score-input" bind:value={scoreInput}>
        </div>
        <div class="input-items">
            <label for="goals-input" style="font-size: 150%;">Goals</label>
            <input type="number" id="goals-input" bind:value={goalsInput}>
        </div>
        <div class="input-items">
            <label for="assists-input" style="font-size: 150%;">Assists</label>
            <input type="number" id="assists-input" bind:value={assistsInput}>
        </div>
        <div class="input-items">
            <label for="saves-input" style="font-size: 150%;">Saves</label>
            <input type="number" id="saves-input" bind:value={savesInput}>
        </div>
        <div class="input-items">
            <label for="shots-input" style="font-size: 150%;">Shots</label>
            <input type="number" id="shots-input" bind:value={shotsInput}>
        </div>
    </div>
    <div class="input-bottom-half">
      <div class="car-input"> 
          <div class="car-selection" style="display: flex; flex-direction: column; align-items: center; padding: 20px 20px 0 20px;">
              <img src="src/images/Octane-img.png" style="width: 100%; height: 100%;" on:click={() => carSelected('octane')} id="octanePic" class="car-img" style:border={carBorder}>
              <h3 style="position: relative;bottom: 10px;" id="octane-text" class="octane-text" >Octane</h3>
          </div>
          <div class="car-selection" style="display: flex; flex-direction: column; align-items: center; padding: 20px 20px 0 0px;">
              <img src="src/images/Fennec-img.png" style="width: 100%; height: 100%;" on:click={() => carSelected('fennec')} id="fennecPic" class="car-img" style:border={carBorder}>
              <h3 style="position: relative;bottom: 10px;" id="fennec-text" class="fennec-text" >Fennec</h3>  
          </div>
          <div class="car-selection" style="display: flex; flex-direction: column; align-items: center; padding: 20px 20px 0 0px;">           
              <img src="src/images/Dominus-img.png" style="width: 100%; height: 100%;" on:click={() => carSelected('dominus')} id="dominusPic" class="car-img" style:border={carBorder}>
              <h3 style="position: relative;bottom: 10px;" id="dominus-text" class="dominus-text" >Dominus</h3>
          </div>
          <div class="car-selection" style="display: flex; flex-direction: column; align-items: center; padding: 20px 20px 0 0px;">
              <div id="other" class="other" on:click={() => carSelected('other')} style="width: 100%; height: 100%; border: white solid 2px;  border-radius: 20px;" style:border={carBorder}>?</div>
              <h3 style="position: relative;bottom: 10px;" id="other-text" class="other-text" >Other</h3>
          </div>
      </div>
      <div class="gamemode-and-buttons">
        <div class="input-gamemode">
          <h2 id="headerEle" style="font-size: 200%;">Gamemode</h2>
          <div style="display: flex; flex-direction: column; gap: 20px;">
            <div style="display: flex; flex-direction: row; align-items: center; gap: 10px;">
              <input class="gamemode-checkbox" type="checkbox" id="3v3-input" bind:checked={threeVthreeInput}>
              <label id="labels" class="gamemode-checkbox" for="3v3-input" style="font-size: 150%;" >3v3</label>
            </div>
            <div style="display: flex; flex-direction: row; align-items: center; gap: 10px;">
              <input class="gamemode-checkbox" type="checkbox" id="2v2-input" bind:checked={twoVtwoInput}>
              <label id="labels" class="gamemode-checkbox" for="2v2-input" style="font-size: 150%;">2v2</label>
            </div>
            <div style="display: flex; flex-direction: row; align-items: center; gap: 10px;">
              <input class="gamemode-checkbox" type="checkbox" id="1v1-input" bind:checked={oneVoneInput}>
              <label id="labels" class="gamemode-checkbox" for="1v1-input" style="font-size: 150%;">1v1</label>
            </div>
          </div>
        </div>
        <div class="buttons">
          <button class="submit-button" on:click={() => submit()}>Submit</button>
          <button class="cancel-button" on:click={() => cancel()}>Cancel</button>
        </div>
        <div id="error" class="error">{errorMessage1}</div>
      </div>
    </div>
  </div>
</main>
<footer id='footer'>
  <p>Owen Richards</p>
  <p id="clock">{clock}</p>
  <p id="timer">{timer}</p>
</footer>

<style>
</style>
