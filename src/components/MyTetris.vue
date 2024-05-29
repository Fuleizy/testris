<template>
  <h1>Score: {{ score }}</h1>
  <canvas ref="canvas"></canvas>
  <br />
  <bold>Instructions:</bold>
  <br />
  <bold>'a': move piece left</bold>
  <br />
  <bold>'d': move piece right</bold>
  <br />
  <bold>'s': move piece down</bold>
  <br />
  <bold>'w': rotate piece</bold>
  <br />
</template>

<script setup>
import { ref, onMounted } from 'vue'
const score = ref(0)
onMounted(() => {
  const canvas = document.querySelector('canvas')
  const context = canvas.getContext('2d')

  const BLOCK_SIZE = 20
  const BOARD_WIDTH = 14
  const BOARD_HEIGHT = 30

  canvas.width = BLOCK_SIZE * BOARD_WIDTH
  canvas.height = BLOCK_SIZE * BOARD_HEIGHT

  context.scale(BLOCK_SIZE, BLOCK_SIZE)

  const board = Array(BOARD_HEIGHT)
    .fill()
    .map(() => Array(BOARD_WIDTH).fill(0))

  const PIECES = [
    [
      [1, 1],
      [1, 1]
    ],
    [[1, 1, 1, 1]],
    [
      [0, 1, 0],
      [1, 1, 1]
    ],
    [
      [1, 1, 0],
      [0, 1, 1]
    ],
    [
      [0, 1, 1],
      [1, 1, 0]
    ],
    [
      [1, 0, 0],
      [1, 1, 1]
    ],
    [
      [0, 0, 1],
      [1, 1, 1]
    ]
  ]

  const piece = {
    position: { x: BOARD_WIDTH / 2 - 1, y: 0 },
    shape: PIECES[Math.floor(Math.random() * PIECES.length)]
  }

  function getRandomPiece() {
    // get random shape
    piece.shape = PIECES[Math.floor(Math.random() * PIECES.length)]
    // reset position
    piece.position.x = BOARD_WIDTH / 2 - 1
    piece.position.y = 0

    for (var i = 0; i < Math.floor(Math.random() * 2); i++) {
      rotatePiece()
    }

    if (checkCollision()) {
      board.forEach((row) => row.fill(0))
      window.alert('Game over!! Sorry!!')
      score.value = 0
    }
  }

  let dropCounter = 0
  let lastTime = 0
  function update(time = 0) {
    const deltaTime = time - lastTime
    lastTime = time

    dropCounter += deltaTime
    if (dropCounter > 1000) {
      piece.position.y++
      if (checkCollision()) {
        piece.position.y--
        solidifyPiece()
        removeRows()
      }
      dropCounter = 0
    }

    draw()
    window.requestAnimationFrame(update)
  }

  function draw() {
    context.fillStyle = '#000'
    context.fillRect(0, 0, canvas.width, canvas.height)

    board.forEach((row, y) => {
      row.forEach((value, x) => {
        if (value === 1) {
          context.fillStyle = 'yellow'
          context.fillRect(x, y, 1, 1)
        }
      })
    })

    piece.shape.forEach((row, y) => {
      row.forEach((value, x) => {
        if (value) {
          context.fillStyle = 'red'
          context.fillRect(x + piece.position.x, y + piece.position.y, 1, 1)
        }
      })
    })
  }

  function rotatePiece() {
    piece.shape = piece.shape[0].map((val, index) => piece.shape.map((row) => row[index]).reverse())
    console.log(piece.position.x, piece.shape[0].length)
    if (piece.position.x + piece.shape[0].length > BOARD_WIDTH) {
      piece.position.x = BOARD_WIDTH - piece.shape[0].length
    }
    if (piece.position.x < 0) {
      piece.position.x = 0
    }
  }

  document.addEventListener('keydown', (event) => {
    if (event.key === 'a') {
      piece.position.x--
      if (checkCollision()) {
        piece.position.x++
      }
    }
    if (event.key === 'd') {
      piece.position.x++
      if (checkCollision()) {
        piece.position.x--
      }
    }
    if (event.key === 's') {
      piece.position.y++
      if (checkCollision()) {
        piece.position.y--
        solidifyPiece()
        removeRows()
      }
    }
    if (event.key === 'w') {
      rotatePiece()
    }
  })

  function checkCollision() {
    return piece.shape.find((row, y) => {
      return row.find((value, x) => {
        return value !== 0 && board[y + piece.position.y]?.[x + piece.position.x] !== 0
      })
    })
  }

  function solidifyPiece() {
    piece.shape.forEach((row, y) => {
      row.forEach((value, x) => {
        if (value === 1) {
          board[y + piece.position.y][x + piece.position.x] = 1
        }
      })
    })
    getRandomPiece()
  }

  function removeRows() {
    const rowsToRemove = []

    board.forEach((row, y) => {
      if (row.every((value) => value === 1)) {
        rowsToRemove.push(y)
      }
    })

    score.value += rowsToRemove.length * rowsToRemove.length * 10

    rowsToRemove.forEach((y) => {
      board.splice(y, 1)
      const newRow = Array(BOARD_WIDTH).fill(0)
      board.unshift(newRow)
    })
  }
  update()
})
</script>
