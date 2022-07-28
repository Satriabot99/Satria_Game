'use strict';

document.addEventListener('DOMContentLoaded', () => {

    alert('🙋‍♂️ Satria Pamungkas! 🤪  ');

    alert('The goal of the game is to move the squares by clicking on them, arrange them by numbers using a free window, while taking the least number of steps.');
    
    const appPuzzle = () => {
      
      // game
      const $appNode = document.getElementById('app__puzzle');
      const $appNodeLock = document.getElementById('app__puzzle--lock');
      const $appShuffle = document.getElementById('app__shuffle');
      const $appCounter = document.getElementById('app__counter');
    
      const $appResultCount = document.getElementById('app__result--count'); 
      const $appResultTime = document.getElementById('app__result--time'); 
    
      const $appItemsNodes = Array.from($appNode.querySelectorAll('.app__puzzle--btn'));
      const $appShuffleBtn = document.querySelector('.app__puzzle--shuffle');
      const $appRules = document.querySelector('.app__rules');
      const $appBox = document.querySelector('.app__box');
      const $appPublick = document.querySelector('.app__puzzle--public');
      const $appActual = document.querySelector('.app__actual');
    
      // audio
      const $validAudio = new Audio('https://www.mboxdrive.com/clickBtnValid.mp3');
      const $noValidAudio = new Audio('https://www.mboxdrive.com/noclick.mp3');
      const $wonAudio = new Audio('https://www.mboxdrive.com/wongame.mp3');
      const $startAudio = new Audio('https://www.mboxdrive.com/start.mp3');
    
    
      // preloader
      const $preloader = document.querySelector('.preloader');
    
      // time
      const $minute = document.getElementById('minute');
      const $second = document.getElementById('second');
      const $millisecond = document.getElementById('millisecond');
    
      const winFlatArr = new Array(16).fill(0).map((_item, index) => index + 1);
    
      const data = {
        count: 0,
        maxCount: 9999,
        time: '00:00:00',
        ms: 0,
        limit: 3599990,
      };
    
      // variables
      const blankNumber = 16;
      const countItems = 16;
      const countLine = 4;
      const maxShuffleCount = 100;
    
      let shuffled = false;
      let matrix = [];
      let blockedCoords = null;
      let shuffleCount = null;
      let timer = null;
    
      let minute = 0;
      let second = 0;
      let millisecond = 0;
      let interval = null;
    
      $appCounter.textContent = data.count;
    
      /** Reset time */
      const resetTimer = () => {
        minute = 0;
        second = 0;
        millisecond = 0;
        $millisecond.textContent = addZero(millisecond);
        $second.textContent = addZero(second);
        $minute.textContent = addZero(minute);
      };
    
      const timeWonGame = () => `${addZero(minute)}:${addZero(second)}:${addZero(millisecond)}`;
    
      const noValidClick = () => {
        if (window.navigator && window.navigator.vibrate) {
          navigator.vibrate(100);
          audioPlay($noValidAudio);
        }
        else {
          audioPlay($noValidAudio);
        }
      };
    
      /** Zero */
      const addZero = n => n < 10 ? '0' + n : n;
    
      /** Hide */
      const addClass = (elem, name) => elem.classList.add(name);
    
      /** Show */
      const removeClass = (elem, name) => elem.classList.remove(name);
    
      const notificationResult = text => {
        $appPublick.textContent = text;
        addClass($appPublick, 'open');
        setTimeout(() => removeClass($appPublick, 'open'), 5000);
      };
    
      const totalCount = () => {
        if (data.count < data.maxCount) {
          data.count++;
          $appCounter.textContent = data.count;
        }
      };
    
      const removePreloader = () => {
        if (!$preloader.classList.contains('hide')) {
          addClass($preloader, 'hide');
        }
      };
    
      const checkCountItems = () => {
        if ($appItemsNodes.length < countItems) {
          throw new Error(`Должно быть ровно ${countItems} items in HTML`);
        }
      };
    
      const hideItemsLast = () => {
        $appItemsNodes[countItems - 1].style.display = 'none';
      };
    
      const checkDatasetItems = () => $appItemsNodes.map(item => Number(item.dataset.matrixId));
    
      const removeDisabled = () => {
        if (!!$appShuffle.getAttribute('disabled')) {
          $appShuffle.removeAttribute('disabled');
        }
      };
    
      const removeNotificationRecord = () => {
        if ($appPublick.classList.contains('open')) {
          removeClass($appPublick, 'open');
        }
      };
    
      const removeAnimateOpacity = () => {
          if ($appActual.classList.contains('animate-opacity')) {
            removeClass($appActual, 'animate-opacity');
          }
      };
      
      const animateShuffleButton = () => {
          addClass($appShuffleBtn, 'animate-btn');
          setTimeout(() => {
              removeClass($appShuffleBtn, 'animate-btn');
          }, 1150);
      };
    
      const audioPlay = audio => audio.play();
    
      const resetCurrentTime = elem => elem.currentTime = 0;
    
      /** Minutes */
      const clearTimer = () => {
        if (minute === 59 && second === 59 && millisecond === 99) {
          clearInterval(interval);
          init();
        }
      };
    
      /** Seconds */
      const timerSeconds = () => {
        if (second > 59) {
          minute++;
          $minute.textContent = addZero(minute);
          second = 0;
          $second.textContent = addZero(second);
        }
    
        clearTimer();
      };
    
        /** Milliseconds */
        const timerMilliseconds = () => {
          if (millisecond > 99) {
            second++;
            $second.textContent = addZero(second);
            millisecond = 0;
            $millisecond.textContent = addZero(millisecond);
          }
          timerSeconds();
        };
    
      /** Timer */
      const startTimer = () => {
        millisecond++;
        $millisecond.textContent = addZero(millisecond);
        timerMilliseconds();
      };
    
      /** Matrix */
      const getMatrix = arr => {
        const matrix = [[], [], [], []];
        let y = 0,
            x = 0;
    
        for (let item of arr) {
          if (x >= countLine) {
            y++;
            x = 0;
          }
          matrix[y][x] = item;
          x++;
        }
    
        return matrix;
      };
    
      /** Show won */
      const isWon = matrix => {
        const flatMatrix = matrix.flat();
        for (let i = 0; i < winFlatArr.length; i++) {
          if (flatMatrix[i] !== winFlatArr[i]) {
            return false;
          }
        }
        return true;
      };
    
      const limitTimeGame = () => {
        if (data.ms === data.limit) {
          notificationResult('The time limit has been reached!');
          addClass($appActual, 'animate-opacity');
          resetTimer();
          data.count = 0;
          data.time = '00:00:00';
          audioPlay($wonAudio);
        }
      };
      
      const checkResult = () => {
          if (data.count > 0) {
              audioPlay($wonAudio);
              addClass($appActual, 'animate-opacity');
              notificationResult('Your result!');
          }
      };
    
      const addWonClass = () => {
        if (isWon(matrix)) {
    
          data.time = timeWonGame();
          data.ms = (s => 1E1 * s[2] + 1E3 * s[1] + 6E4 * s[0])(data.time.split(':'));
    
          removeDisabled();
          limitTimeGame();
          checkResult();
          
          clearInterval(interval);
          resetCurrentTime($wonAudio);
      
          $appResultCount.textContent = data.count;
          $appResultTime.textContent = data.time;
          
          removeClass($appNodeLock, 'hide');
          addClass($appBox, 'hide');
          removeClass($appRules, 'hide');
    
          setTimeout(() => removeClass($appNode, 'won'), 70);
        }
        else {
          setTimeout(() => addClass($appNode, 'won'), 70);
        }
      };
    
      /** Position */
      const setNodeStyles = (node, x, y) => {
        const shiftPs = 100;
        node.style.transform = `translate3D(${x * shiftPs}%, ${y * shiftPs}%, 0)`;
      };
    
      const setPositionItems = matrix => {
    
        for (let y = 0; y < matrix.length; y++) {
          for (let x = 0; x < matrix[y].length; x++) {
            const value = matrix[y][x];
            const node = $appItemsNodes[value - 1];
            
            setNodeStyles(node, x, y);
          }
        }
    
        addWonClass();
      };
    
      const findCoordinatesByNumber = (number, matrix) => {
        for (let y = 0; y < matrix.length; y++) {
          for (let x = 0; x < matrix[y].length; x++) {
            if (matrix[y][x] === number) return {y, x};
          }
        }
    
        return null;
      };
    
      const isValidForSwap = (coord01, coord02) => {
        const diffX = Math.abs(coord01.x - coord02.x);
        const diffY = Math.abs(coord01.y - coord02.y);
    
        return (
          (diffX === 1 || diffY === 1) && 
          (coord01.x === coord02.x || coord01.y === coord02.y)
        );
      };
    
      const swapItems = (coords01, coords02, matrix) => {
        const tempCoords = matrix[coords01.y][coords01.x];
        matrix[coords01.y][coords01.x] = matrix[coords02.y][coords02.x];
        matrix[coords02.y][coords02.x] = tempCoords;
      };
    
      const getСoordinatesNumber = (btnCoords, blankCoords) => {
        const isValid = isValidForSwap(btnCoords, blankCoords);
      
        if (isValid) {
          totalCount();
          resetCurrentTime($validAudio);
          audioPlay($validAudio);
          swapItems(btnCoords, blankCoords, matrix);
          setPositionItems(matrix);
        }
        else {
          resetCurrentTime($noValidAudio);
          noValidClick();
        }
      };
    
      /** Shuffle */
      const findValidCoords = ({ blankCoords, matrix, blockedCoords }) => {
        const validCoords = [];
    
        for (let y = 0; y < matrix.length; y++) {
          for (let x = 0; x < matrix[y].length; x++) {
            if (isValidForSwap({ x, y }, blankCoords)) {
              if (!blockedCoords || !(blockedCoords.x === x && blockedCoords.y === y)) {
                validCoords.push({ x, y });
              }
            }
          }
        }
    
        return validCoords;
      };
    
      const randomSwap = matrix => {
        const blankCoords = findCoordinatesByNumber(blankNumber, matrix);
    
        const validCoords = findValidCoords({ blankCoords, matrix, blockedCoords });
    
        const swapCoords = validCoords[
          Math.floor(Math.random() * validCoords.length)
        ];
        
        swapItems(blankCoords, swapCoords, matrix);
        blockedCoords = blankCoords;
      };
    
      const startingGame = () => {
        timer = setInterval(() => {
          randomSwap(matrix);
          setPositionItems(matrix);
          $appShuffle.setAttribute('disabled', true);
    
          shuffleCount += 1;
          
          if (shuffleCount >= maxShuffleCount) {
            addClass($appNodeLock, 'hide');
            addClass($appRules, 'hide');
            removeClass($appBox, 'hide');
    
            resetTimer();
            clearInterval(timer);
            clearInterval(interval);
    
            interval = setInterval(startTimer, 10);
            shuffled = false;
          }
        }, 20);
      };
    
      const clickShuffleItems = () => {
        if (shuffled) return;
    
        shuffled = true;
        shuffleCount = 0;
    
        data.count = 0;
        $appCounter.textContent = data.count;
    
        removeNotificationRecord();
        removeAnimateOpacity();
        
        clearInterval(timer);
    
        if (shuffleCount === 0) {
          animateShuffleButton();
          resetCurrentTime($startAudio);
          audioPlay($startAudio);
          startingGame();
        }
      };
    
      /** Change position by click */
      const clickTargetItems = ({ target }) => {
        if (shuffled) return;
        const btnNode = target.closest('.app__puzzle--btn');
        if (!btnNode) return;
    
        const btnNumber = Number(btnNode.dataset.matrixId);
    
        const btnCoords = findCoordinatesByNumber(btnNumber, matrix);
        const blankCoords = findCoordinatesByNumber(blankNumber, matrix);
    
        getСoordinatesNumber(btnCoords, blankCoords);
      };
    
      /** Events */
      $appNode.addEventListener('click', clickTargetItems);
      $appShuffle.addEventListener('click', clickShuffleItems);
    
      /** Start */
      const init = () => {
        checkCountItems();
        hideItemsLast();
        const matrixId = checkDatasetItems();
        matrix = getMatrix(matrixId);
        setPositionItems(matrix);
      };
    
      setTimeout(removePreloader, 2000);
      init();
    };
    
    appPuzzle();
});