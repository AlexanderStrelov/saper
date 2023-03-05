<template>
    <div class="game" :class="gameState">
        <div class="toolbar options border">
            <div>Mines </div>
            <div class="mines_quantity">
                <input type="number" v-model="options.minesNumber">
            </div>
            <div>Width</div>
            <div class="game_width">
                <input type="number" v-model="options.gameWidth">
            </div>
            <div>Height</div>
            <div class="game_height">
                <input type="number" v-model="options.gameHeight">
            </div>
            <div>Timer min</div>
            <div class="game_timer">
                <input type="number" v-model="options.gameInterval">
            </div>
            <div class="buttons">
                <button @click="applyOptions">Confirm</button>
            </div>
        </div>
        <div class="main">
            <div class="timer_min">
                <img :src="imageMinutes.firstImageMinutes">
                <img :src="imageMinutes.secondImageMinutes">
                <img :src="imageMinutes.thirdImageMinutes">
            </div>
            <div class="title">
                <div class="smile" :class="smileRender" @click="resetGame"></div>
            </div>
            <div class="timer_sec">
                <img :src="imageSeconds.imageSecondsFirst">
                <img :src="imageSeconds.imageSecondsSecond">
                <img :src="imageSeconds.imageSecondsThird">
            </div>
        </div>
        <div class="board">
            <div v-for="(mapRow, index) in map" :key="index" class="row">
                <div class="cell" v-for="(cell, index) in mapRow" :key="index" :style="{ width: (100 / gameWidth) + '%' }"
                    :class="cellClass(cell)"
                    @mousedown.left="onCellClick(cell), gameState === 'play' ? activeCell = 1 : activeCell = 0"
                    @mouseup.left="activeCell = 0" @mousedown.right="onCellRightClick(cell)" @contextmenu.prevent>
                    <div v-if="cell.isOpen && !cell.isMine && cell.mines" class="cell__text">
                        <!-- {{ cell.mines }} -->
                    </div>
                </div>
            </div>

        </div>
    </div>
</template>

<script>


class GameCell {
    constructor(x, y) {
        this.x = x;
        this.y = y;
        this.mines = 0;
        this.isMine = false;
        this.isOpen = false;
        this.flag = false;
        this.question = false;
    }

    swap(cell) {
        this.swapVals(cell, 'isMine');
        this.swapVals(cell, 'mines');
        this.swapVals(cell, 'isOpen');
        this.swapVals(cell, 'flag');
        this.swapVals(cell, 'question');
    }
    swapVals(cell, field) {
        const val = this[field];
        this[field] = cell[field];
        cell[field] = val;
    }
}
export default {
    data() {
        return {
            gameWidth: 0,
            gameHeight: 0,
            minesNumber: 0,
            gameInterval: 0,
            gameTimer: 0,
            separatedInterval: {
                firstSeparatedInterval: 0,
                secondSeparatedInterval: 0,
                thirdSeparatedInterval: 0,
            },
            imageMinutes: {
                firstImageMinutes: require(`../assets/img/0.png`),
                secondImageMinutes: require(`../assets/img/0.png`),
                thirdImageMinutes: require(`../assets/img/0.png`),
            },
            imageSeconds: {
                imageSecondsFirst: require(`../assets/img/0.png`),
                imageSecondsSecond: require(`../assets/img/0.png`),
                imageSecondsThird: require(`../assets/img/0.png`),
            },
            separatedTimer: {
                firstSeparatedTimer: 0,
                secondSeparatedTimer: 0,
                thirdSeparatedTimer: 0,
            },
            minutes: 0,
            seconds: 0,
            map: [],
            options: {
                gameWidth: 16,
                gameHeight: 16,
                minesNumber: 40,
                gameInterval: 40,
            },
            gameState: 'init',
            cellsOpened: 0,
            flagsCount: 0,
            activeCell: 0,
            questionCount: 0,
            cellClick: 0,
        };
    },
    computed: {
        cellsClosed() {
            return (this.gameHeight * this.gameWidth) - this.cellsOpened;
        },
        smileRender() {
            return {
                'smile_game_over': this.gameState === 'fail',
                'smile_win': this.gameState === 'win',
                'smile_horor': this.activeCell === 1,
            }
        },
    },
    watch: {
        gameTimer(value) {
            if (String(value).length < 2) {
                this.separatedInterval.firstSeparatedInterval = 0;
                this.separatedInterval.secondSeparatedInterval = 0;
                this.separatedInterval.thirdSeparatedInterval = Number(String(value)[0]);
            }
            if (String(value).length < 3 && String(value).length >= 2) {
                this.separatedInterval.firstSeparatedInterval = 0;
                this.separatedInterval.secondSeparatedInterval = Number(String(value)[0]);
                this.separatedInterval.thirdSeparatedInterval = Number(String(value)[1]);
            }
            if (String(value).length < 4 && String(value).length >= 3) {
                this.separatedInterval.firstSeparatedInterval = Number(String(value)[0]);
                this.separatedInterval.secondSeparatedInterval = Number(String(value)[1]);
                this.separatedInterval.thirdSeparatedInterval = Number(String(value)[2]);
            }
        },
        gameInterval(value) {
            if (String(value).length < 2) {
                this.separatedTimer.firstSeparatedTimer = 0;
                this.separatedTimer.secondSeparatedTimer = 0;
                this.separatedTimer.thirdSeparatedTimer = Number(String(value)[0]);
            }
            if (String(value).length < 3 && String(value).length >= 2) {
                this.separatedTimer.firstSeparatedTimer = 0;
                this.separatedTimer.secondSeparatedTimer = Number(String(value)[0]);
                this.separatedTimer.thirdSeparatedTimer = Number(String(value)[1]);
            }
            if (String(value).length < 4 && String(value).length >= 3) {
                this.separatedTimer.firstSeparatedTimer = Number(String(value)[0]);
                this.separatedTimer.secondSeparatedTimer = Number(String(value)[1]);
                this.separatedTimer.thirdSeparatedTimer = Number(String(value)[2]);
            }

        },
    },
    methods: {
        countDownTimer() {
            if (this.gameInterval <= 0) {
                this.gameOver();
                return;
            }
            if (this.gameInterval > 0 && this.gameState === 'play') {
                let timer = setTimeout(() => {
                    if (this.gameInterval === 0) {
                        clearTimeout(timer)
                        return;
                    }
                    this.gameInterval -= 1
                    this.countDownTimer()
                }, 1000 * 60)
            }
        },
        countUpTimer() {
            let interval = setInterval(() => {
                if (this.gameState === 'int' || this.gameState === 'win') {
                    clearInterval(interval);
                    return;
                } if (this.gameState === 'fail') {
                    clearInterval(interval);
                    this.gameTimer = 999;
                    return;
                } else {
                    this.gameTimer += 1;
                }
            }, 1000)
        },
        applyOptions() {
            for (let opt in this.options)
                this[opt] = this.options[opt];

            this.resetGame();
        },
        clearBoard() {
            this.map = [];
            for (let y = 0; y < this.gameHeight; y++) {
                this.map.push([]);
                for (let x = 0; x < this.gameWidth; x++) {
                    this.map[y].push(new GameCell(x, y));
                }
            }
        },
        seedMines(exeptCell) {
            const cellNumber = this.gameWidth * this.gameHeight;
            const exeptNumber = exeptCell.y * this.gameWidth + exeptCell.x

            for (let i = 0; i < this.minesNumber; i++)
                if (i !== exeptNumber)
                    this.cellLine(i).isMine = true;

            for (let i = 0; i < cellNumber - 1; ++i)
                if (i !== exeptNumber) {
                    const exeprtion = i < exeptNumber;

                    let target = i + Math.floor(Math.random() * (cellNumber - i) + (exeprtion ? -1 : 0));

                    if (exeprtion && target >= exeptNumber)
                        target++;

                    const cellN = this.cellLine(i);
                    const cellTarget = this.cellLine(target);

                    cellN.swap(cellTarget);
                }
        },
        cellLine(i) {
            return this.cell(i % this.gameWidth, Math.floor(i / this.gameWidth))
        },
        cell(x, y) {
            return (x >= 0 && y >= 0 && x < this.gameWidth && y < this.gameHeight)
                ? this.map[y][x]
                : null;
        },
        cellClass(cell) {
            return cell.isOpen
                ? [cell.isMine ? 'mine' : 'digit-' + cell.mines]
                : ['closed', cell.flag ? ['flag', cell.question ? 'question' : ''] : ''];
        },
        resetGame() {
            this.gameState = 'int';
            this.cellsOpened = 0;
            this.flagsCount = 0;
            this.questionCount = 0;
            this.gameInterval = this.options.gameInterval
            clearInterval(this.countUpTimer);
            this.gameTimer = 0;
            this.clearBoard();
            // this.seedMines();
        },
        onCellClick(cell) {
            this.checkInit(cell);

            if (this.gameState === 'play' && !cell.isOpen) {
                if (cell.isMine) {
                    this.gameOver();
                } else {
                    try {
                        this.openCell(cell);

                        if (Number(this.cellsClosed) === Number(this.minesNumber)) {
                            this.winGame();
                        }
                    }
                    catch {
                        this.gameOver();
                    }
                }
            }
        },
        onCellRightClick(cell) {
            this.checkInit(cell);
            if (this.gameState === 'play') {

                if (!cell.isOpen && !cell.flag) {
                    this.setCellFlag(cell, !cell.flag);
                    return;
                } else if (!cell.isOpen && cell.flag) {
                    this.setCellQuestion(cell, !cell.question);
                    if (cell.flag && !cell.question && !cell.isOpen) {
                        this.clearAllPoints(cell, false, false)
                        return;
                    }
                }


            }
        },
        setCellFlag(cell, flag) {
            if (flag !== cell.flag) {
                cell.flag = flag;
                this.flagsCount += flag ? 1 : -1;

            }

        },
        setCellQuestion(cell, question) {
            if (question !== cell.question) {
                cell.question = question;
                this.questionCount += question ? 1 : -1;
            }
        },
        clearAllPoints(cell, question, flag) {
            cell.question = question;
            cell.flag = flag;
            this.questionCount += question ? 1 : -1;
            this.flagsCount += flag ? 1 : -1;
        },
        openCell(cell) {

            if (cell && !cell.isOpen) {
                this.setCellOpen(cell)

                if (cell.isMine) {
                    throw "BANG!";
                }
                if (cell.mines === 0) {
                    this.walkAround(cell, neighbour => this.openCell(neighbour));
                }
            }

        },
        setCellOpen(cell) {
            if (!cell.isOpen) {
                cell.isOpen = true;
                this.cellsOpened++;
                this.setCellFlag(cell, false);

                if (!cell.isMine) {
                    cell.mines = this.countMinesAround(cell);
                }
            }
        },
        checkInit(cell) {
            if (this.gameState === 'int') {
                this.gameState = 'play';
                clearInterval(this.countUpTimer);
                this.countUpTimer();
                this.countDownTimer();
                this.seedMines(cell);
            }
        },
        gameOver() {
            this.gameState = 'fail';
            clearInterval(this.countUpTimer);
            this.openAllCells();
        },
        winGame() {
            this.gameState = 'win';
            this.openAllCells();

        },
        openAllCells() {
            for (let y = 0; y < this.map.length; ++y)
                for (let x = 0; x < this.map.length; ++x) {
                    let cell = this.cell(x, y);
                    cell.isOpen = true;
                    if (!cell.isMine) {
                        cell.mines = this.countMinesAround(cell);
                    }
                }
        },
        countMinesAround(cell) {
            return this.countCellsAround(cell, neighbour => neighbour.isMine)
        },
        countCellsAround(cell, filter) {
            let num = 0;

            this.walkAround(cell, neighbour => num += filter(neighbour) ? 1 : 0);
            return num;
        },
        walkAround(cell, callback) {
            let neighbour;

            for (let dx = -1; dx < 2; ++dx)
                for (let dy = -1; dy < 2; ++dy)
                    if ((dx || dy) && (neighbour = this.cell(cell.x + dx, cell.y + dy))) {
                        callback(neighbour);
                    }
        },
    },
    created() {
        this.applyOptions();
        /** картинки для таймера */
        this.$watch(() => this.separatedTimer.firstSeparatedTimer, (val) => {
            if (val === 0 && this.gameState === 'int') {
                this.imageMinutes.firstImageMinutes = require(`../assets/img/0.png`);
            }
            this.imageMinutes.firstImageMinutes = require(`../assets/img/${val}.png`);
        })

        this.$watch(() => this.separatedTimer.secondSeparatedTimer, (val) => {
            if (val === 0 && this.gameState === 'int') {
                this.imageMinutes.secondImageMinutes = require(`../assets/img/0.png`);
            }
            this.imageMinutes.secondImageMinutes = require(`../assets/img/${val}.png`);
        })

        this.$watch(() => this.separatedTimer.thirdSeparatedTimer, (val) => {
            if (val === 0 && this.gameState === 'int') {
                this.imageMinutes.thirdImageMinutes = require(`../assets/img/0.png`);
            }
            this.imageMinutes.thirdImageMinutes = require(`../assets/img/${val}.png`);
        })
        /** картинки для секундамера */
        this.$watch(() => this.separatedInterval.firstSeparatedInterval, (val) => {
            if (val === 0 && this.gameState === 'int') {
                this.imageSeconds.imageSecondsFirst = require(`../assets/img/0.png`);
            }
            this.imageSeconds.imageSecondsFirst = require(`../assets/img/${val}.png`);
        })

        this.$watch(() => this.separatedInterval.secondSeparatedInterval, (val) => {
            if (val === 0 && this.gameState === 'int') {
                this.imageSeconds.imageSecondsSecond = require(`../assets/img/0.png`);
            }
            this.imageSeconds.imageSecondsSecond = require(`../assets/img/${val}.png`);
        })

        this.$watch(() => this.separatedInterval.thirdSeparatedInterval, (val) => {
            if (val === 0 && this.gameState === 'int') {
                this.imageSeconds.imageSecondsThird = require(`../assets/img/0.png`);
            }
            this.imageSeconds.imageSecondsThird = require(`../assets/img/${val}.png`);
        })

    },

}
</script>
<style>
.border {
    border: transparent;
    border-radius: 5px;
}

.game {
    max-width: 600px;
    margin: auto auto;
    background-color: #ccc;
    padding: 15px;
    border: transparent;
    border-radius: 20px;
}

.toolbar {
    display: flex;
    justify-content: space-between;
    background-color: #2a07aa;
    color: #fff;
    padding: 10px 15px;
    font-size: 19px;
    line-height: 25px;
}

.main {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
}

.options {
    font-size: 16px;
    line-height: 25px;
    margin-bottom: 10px;
}

.toolbar input {
    width: 40px;
    text-align: center;
}

.toolbar .buttons {
    min-width: 100px;
    min-height: 25px;
}

.toolbar .buttons button {
    min-width: 100%;
    min-height: 100%;
    cursor: pointer;
    box-sizing: border-box;
}

.board {
    margin-top: 10px;
    background-color: white;
    border: 3px solid black;
}

.cell {
    box-sizing: border-box;
    text-align: center;
    border: 1px solid #ccc;
    position: relative;
    cursor: pointer;

    background-position: center;
    background-repeat: no-repeat;
    background-size: 100%;

    font-weight: bold;
}

.row {
    display: flex;
    flex-direction: row;
}

.cell::after {
    content: '';
    display: block;
    padding-bottom: 100%;
}

.cell__text {
    position: absolute;
    width: 100%;
    height: 100%;

    display: flex;
    flex-direction: column;
    justify-content: center;
    align-content: center;
}

.cell.mine {
    background-image: url('../assets/img/cell-bomb.png');
}

.cell.closed {
    background-image: url('../assets/img/cell.png');
}

.cell.closed:active {
    background-image: url('../assets/img/cell-hover.png');
}

.cell.flag {
    background-image: url('../assets/img/cell-flag.png');
}

.cell.question {
    background-image: url('../assets/img/cell-question.png');
}

.game.fail.cell.mine {
    background-image: url('../assets/img/cell-bomb-dead.png');
}

.digit-0 {
    background-image: url('../assets/img/cell-hover.png');
}

.digit-1 {
    background-image: url('../assets/img/cell-1.png');
}

.digit-2 {
    background-image: url('../assets/img/cell-2.png');
}

.digit-3 {
    background-image: url('../assets/img/cell-3.png');
}

.digit-4 {
    background-image: url('../assets/img/cell-4.png');
}

.digit-5 {
    background-image: url('../assets/img/cell-5.png');
}

.digit-6 {
    background-image: url('../assets/img/cell-6.png');
}

.digit-7 {
    background-image: url('../assets/img/cell-7.png');
}

.digit-8 {
    background-image: url('../assets/img/cell-8.png');
}

.smile {
    width: 40px;
    height: 40px;
    background-image: url('../assets/img/smile.png');
    background-repeat: no-repeat;
    background-position: center;
    background-size: 100%;
    transition: all 0.1s ease-in-out;
}

.smile:active {
    background-image: url('../assets/img/smile-plug.png');
}

.smile_game_over {
    background-image: url('../assets/img/smile-loose.png');
}

.smile_horor {
    background-image: url('../assets/img/bomb-smile.png');
}

.smile_win {
    background-image: url('../assets/img/smile-win.png');
}

.timer_min,
.timer_sec {
    display: flex;
    flex-direction: row;

}
</style>