<script setup>
  import {onMounted, ref} from "vue"

  var place = ref(0);
  var countSteps = 0
  var currentPos = null;
  var currentEvent = null;
  var started = 0;
  var defaultPos = []
  var posInArr = []
  var currentBoard = []
  var selectedPiece = null;
  let avMoves = []

  const pawnMove=(currentPos, pos, selectedPiece)=>{
    // By finding default (row) position of pawn can determine whether it is moving forward or backward
    let colorOfPiece = selectedPiece[0];
    let returnValue = true;
    let col = parseInt(String(currentPos)[0])
    let moveCol = parseInt(String(pos)[0])
    let difference = Math.abs(parseInt(currentPos) - parseInt(pos));
    let defaultPosOfPiece = String(defaultPos[selectedPiece][0])[1]
    if(!((difference == 2) || difference == 1)){
      console.log("Illegal move")
      returnValue = false
    }
    return returnValue
  }

 function getAvailableMoves(currentPos, pos, piece){
  if(parseInt(currentPos) < 11)
    return

  let colorOfPiece = piece[0];
  let col = parseInt(String(currentPos)[0])
  let row = parseInt(String(currentPos)[1])
  let moveCol = parseInt(String(pos)[0])
  let moveRow = parseInt(String(pos)[1])
  let colArr = []
  let rowArr = []
  for(let i=8; i>=1; i--){
    let rowCords = `${col}${i}`;//parseInt(String(col)+String(i));
    let colCords = `${i}${row}`;
    let pieceOnColCords = place.value[ parseInt( posInArr[colCords] ) ].classList[3];
    let pieceOnRowCords = place.value[ parseInt( posInArr[rowCords] ) ].classList[3];
    
    if(parseInt(colCords) !== parseInt(currentPos)){
      if(typeof(pieceOnColCords) === 'undefined'){
        // console.log("con1 ",colCords)
        colArr.push(colCords)
      }
      else if(typeof(pieceOnColCords) !== 'undefined'){
        if(pieceOnColCords[0] !== colorOfPiece){
          colArr.push(colCords)
        }
      }
    }
   
    if(parseInt(rowCords) !== parseInt(currentPos)){
      if(typeof(pieceOnRowCords) === 'undefined'){
        rowArr.push(rowCords)
      }
      else if(typeof(pieceOnRowCords) !== 'undefined'){
        if(pieceOnRowCords[0] !== colorOfPiece) 
          rowArr.push(rowCords)
      }
    }
  }

  // if(col == moveCol){
  //   console.log("move along vertical",rowArr)
  // }else{
  //   console.log("move along horizontal",colArr)

  // }
      

   console.log(colArr.concat(rowArr))
 }

 function getAvailableMovesOfCastle(currentPos, pos, piece){
  let colorOfPiece = piece[0];
  let col = parseInt(String(currentPos)[0])
  let row = parseInt(String(currentPos)[1])
  let moveCol = parseInt(String(pos)[0])
  let moveRow = parseInt(String(pos)[1])
  let limit = 0
  let start = 0
  let validMove = true
  let removeFirstPiece = 0

  if(col == moveCol){
    console.log("vertically")
    limit=(row>moveRow)? row: moveRow
    start=(row<moveRow)? row: moveRow
    
    for(let i=start;i<=limit;i++){
      if(`${col}${i}`!== String(currentPos)){
        if(currentBoard[`${col}${i}`] !== ''){//cannot jump
          console.log(`${col}${i}`,currentBoard[`${col}${i}`],currentBoard[`${col}${i}`][0]=='b')
          validMove = false
          currentBoard[`${col}${i}`][0] !==colorOfPiece ? removeFirstPiece++ : removeFirstPiece;
        }
      }
    }
  }else if(row == moveRow){
    console.log("horizontal")
    limit=(col>moveCol)? col: moveCol;
    start=(col<moveCol)? col: moveCol;

    for(let i=start;i<=limit;i++){
      if(`${i}${row}`!== String(currentPos)){
        if(currentBoard[`${i}${row}`] !== ''){//cannot jump
            console.log(`${i}${row}`,currentBoard[`${i}${row}`],currentBoard[`${i}${row}`][0]=='b')
            validMove = false
            currentBoard[`${col}${i}`][0] !== removeFirstPiece? removeFirstPiece++ : removeFirstPiece;
        }
      }
    }
  }else 
    validMove = false

  console.log(validMove, removeFirstPiece);
  if(removeFirstPiece==1){ // if removing first peice
    validMove = true
  }
  removeFirstPiece = 0;
  return validMove
}

function  getAvailableMovesOfPawn(currentPos,selectedPiece){
  let colorOfPiece = selectedPiece[0];
  let defaultPosOfPiece = 0;// To make sure pawn is moving forward
  let step = ''
  avMoves = []
  if(['wp','bp'].includes(selectedPiece)){
    defaultPosOfPiece = String(defaultPos[selectedPiece][0])
    currentPos = String(currentPos)
    if(currentPos[1]==7){
      defaultPos[colorOfPiece+'pm'] = '-'
      currentBoard[currentPos[0]+6] == ''?avMoves.push(currentPos[0]+6): []
      avMoves.length!==0 && currentBoard[currentPos[0]+5] == ''?avMoves.push(currentPos[0]+5): []
    }else if(currentPos[1]==2){
      defaultPos[colorOfPiece+'pm'] = '+'
      currentBoard[currentPos[0]+3] == ''?avMoves.push(currentPos[0]+3): []
      avMoves.length!==0 && currentBoard[currentPos[0]+4] == ''?avMoves.push(currentPos[0]+4): []
    }else{
        step = (defaultPos[colorOfPiece+'pm'])
        currentBoard[currentPos[0]+parseInt(eval(currentPos[1]+step+1))] == ''?avMoves.push(currentPos[0]+parseInt(eval(currentPos[1]+step+1))): []  //using eval function to tell js that str is math oparation
        avMoves.length!==0 && currentBoard[currentPos[0]+parseInt(eval(currentPos[1]+step+2))] && (currentPos[1]==7 || currentPos[1]==2) == ''?avMoves.push(currentPos[0]+parseInt(eval(currentPos[1]+step+2))): []
    }
    console.log("inside avmoves",avMoves);
  }
}

  const castleMove = (currentPos, pos, selectedPiece) =>{
    let returnValue = true    
    returnValue = getAvailableMovesOfCastle(currentPos, pos, selectedPiece)
    return returnValue
  }

  const getPositionOfAllPieces=()=>{
    currentBoard=[]
    for(let i=1; i<=8; i++){
      for(let j=1; j<=8; j++){
        let cords = `${i}${j}`;
        if(typeof(place.value[ parseInt( posInArr[cords] ) ].classList[3]) !== 'undefined'){
          currentBoard[cords] = place.value[ parseInt( posInArr[cords] ) ].classList[3];
        }else
          currentBoard[String(`${i}${j}`)] = "";
      }
    }
  }

  const movePiece = (e, pos)=>{
    let validMove = true;
    let identifyPiece = e.target.classList[3]
    if( countSteps == 0 && typeof(identifyPiece) === 'undefined')
      return 

    //this code is block for when you have selected piece and clicking on some other piece of same side
    console.log(selectedPiece ,identifyPiece)
    if(selectedPiece !== null && typeof(identifyPiece) !== 'undefined'){
      if(selectedPiece[0] == identifyPiece[0]){
        currentEvent.target.classList.remove('selected')
        countSteps=0;
      }
    }

    countSteps++
    if(countSteps ==1 ){
      currentEvent = e
      getPositionOfAllPieces()
      getAvailableMovesOfPawn(pos, identifyPiece)
      e.target.classList.add('selected')
      currentPos = pos
      selectedPiece = identifyPiece
    }

    if(countSteps == 2 && currentEvent !== null){ //if clicking at same postion return it
      if(currentPos == pos){
        validMove = false
        countSteps = 0
        currentEvent.target.classList.remove('selected')
        return
      }
      
      if(typeof(identifyPiece) !== 'undefined' && selectedPiece[0] === identifyPiece[0]){ //cant remove own piece
        countSteps = 0
        validMove = false
        currentEvent.target.classList.remove('selected')
        return
      }

      // Inital moves of pawn
      if(['wp','bp'].includes(selectedPiece)){
        validMove = avMoves.includes(pos)
      }

      //Elephant
      if(['we','be'].includes(selectedPiece)){
        validMove = castleMove(currentPos, pos, selectedPiece)
      }

      if(validMove){
        if(typeof(identifyPiece) !== undefined){ // piece udavtoy tar
          e.target.classList.remove(identifyPiece)
        }
        currentEvent.target.classList.remove(selectedPiece)
        e.target.classList.add(selectedPiece)
      }
      currentEvent.target.classList.remove('selected')
      countSteps=0
    }
  }

  const posInRefArr=()=>{ 
    for(let i=1; i<9; i++){
      for(let j=1; j<9; j++){
        // console.log(i,j)
        if(i-1 !==0){
          // console.log(((i-1)*8)+j-1," ",i,j)
          posInArr[String(i)+String(j)] = String(((i-1)*8)+j-1)
        }else{
          posInArr[String(i)+String(j)] = String(j-1)
        }
      }
    }
  }

  const addPiece=()=>{
    if(started !==0)
      return

    defaultPos['wp'] = [17,27,37,47,57,67,77,87]
    defaultPos['we'] = [18, 88]
    defaultPos['wh'] = [28, 78]
    defaultPos['wb'] = [38, 68]
    defaultPos['wk'] = [48]
    defaultPos['wq'] = [58]
    defaultPos['bp'] = [12,22,32,42,52,62,72,82]
    defaultPos['be'] = [11,81]
    defaultPos['bh'] = [21,71]
    defaultPos['bb'] = [31,61]
    defaultPos['bq'] = [41]
    defaultPos['bk'] = [51]

    for(let key in defaultPos){
      for(let i=0; i<defaultPos[key].length; i++){
        place.value[posInArr[defaultPos[key][i]]].classList.add(key)
      }
    }
    started =1
  }

  onMounted(()=>{
    posInRefArr()
    addPiece()
  })
</script>

<template>
  <div class="container">
    Board Game 
    <div class="board">
      <div  v-for=" i in 8" :key="i">
        <div v-for="j in 8" class='squares' v-bind:key="j" draggable="true" ref ="place"
        :class="['square_'+String(i)+String(j)],[(i+j)%2 !==0? 'odd':'even']" v-on:click="movePiece($event, String(i)+String(j))">
           {{ i }} {{ j }}
        </div>
      </div>
    </div>
  </div>

</template>

<style scope>
  .board {
    display: flex;
    cursor: grab;
  }
  .squares{
    padding: 1rem;
  }
  .odd{
    background-color: beige;
  }
  .even{
    background-color: grey;
  }
  .selected{
    background-color: #ff7979;
    background-blend-mode: hard-light;
  }
  .bq{
    background-image:url(./assets/bq.png);
    background-size: 100%;
  }

  .bk{
    background-image:url(./assets/bk.png);
    background-size: 100%;
  }

  .bp{
    background-image:url(./assets/bp.png);
    background-size: 100%;
  }

  .bb{
    background-image:url(./assets/bb.png);
    background-size: 100%;
  }

  .bh{
    background-image:url(./assets/bh.png);
    background-size: 100%;
  }

  .be{
    background-image:url(./assets/be.png);
    background-size: 100%;
  }
  /* white */
  .wq{
    background-image:url(./assets/wq.png);
    background-size: 100%;
  }

  .wk{
    background-image:url(./assets/wk.png);
    background-size: 100%;
  }

  .wp{
    background-image:url(./assets/wp.png);
    background-size: 100%;
  }

  .wb{
    background-image:url(./assets/wb.png);
    background-size: 100%;
  }

  .wh{
    background-image:url(./assets/wh.png);
    background-size: 100%;
  }

  .we{
    background-image:url(./assets/we.png);
    background-size: 100%;
  }
</style>