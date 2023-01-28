<script lang="ts">
  import Block from "./Block.svelte";
  export let rowCount = 64;
  export let colCount = 128;

  const emberColors = ["ember1", "ember2", "ember3"];

  const pick = (selection: any[], size: number) => {
    if (selection.length <= size) {
      return selection;
    }
    const selected: any[] = [];
    while (selected.length < size) {
      const idx = Math.floor(Math.random() * selection.length);
      selected.push(selection[idx]);
      selection.splice(idx, 1);
    }
    return selected;
  };

  type ColorMap = { [col: string]: { [row: string]: string } };

  const logCoalEdges: { [col: string]: number } = {};
  const logFlameEdges: { [col: string]: number } = {};

  const addCoalEdge = (col: number, row: number) => {
    if (!logCoalEdges[col] || logCoalEdges[col] < row) {
      logCoalEdges[col] = row;
    }
  };

  const addFlameEdge = (col: number, row: number) => {
    if (!logFlameEdges[col] || logFlameEdges[col] > row) {
      logFlameEdges[col] = row;
    }
  };

  const maxLogHeight = Math.floor(rowCount / 3);
  const averageStepWidth = Math.ceil((colCount - 8) / (maxLogHeight * 2));
  const getStepWidth = () =>
    Math.max(
      1,
      Math.min(
        Math.round(averageStepWidth * 1.75),
        Math.round(averageStepWidth * Math.random() * 2)
      )
    );
  const getLogWidth = () => Math.max(6, Math.round(10 * Math.random()));

  const logs: ColorMap = {};
  const coals: ColorMap = {};
  let embers: ColorMap = {};

  let col = 4;
  let midRow = rowCount - 1;
  while (col < colCount - 4) {
    const stepWidth = getStepWidth();
    for (let step = 0; step < stepWidth; step++) {
      const logWidth = getLogWidth();
      if (col + step >= colCount - 4) {
        break;
      }
      for (
        let row = midRow + Math.floor(logWidth / 2);
        row > midRow - Math.floor(logWidth / 2);
        row--
      ) {
        // from bottom-up, place logs
        if (row >= rowCount) {
          continue;
        }
        if (!logs[col + step]) {
          logs[col + step] = {};
        }
        if (row <= midRow + Math.floor(logWidth / 2) - 3) {
          logs[col + step][row] = "log-dark";
          addFlameEdge(col + step, row);
        } else {
          logs[col + step][row] = "log-light";
          addCoalEdge(col + step, row);
        }
      }
    }
    if (col > 4 && midRow === 0) {
      break;
    }
    col += stepWidth;
    if (
      col < colCount / 2 &&
      midRow > rowCount - maxLogHeight &&
      midRow < rowCount
    ) {
      midRow--;
    } else {
      midRow++;
    }
  }

  for (let col in logCoalEdges) {
    // do the coals
    if (logCoalEdges[col] < rowCount - 1) {
      if (!coals[col]) {
        coals[col] = {};
      }
      coals[col][logCoalEdges[col] + 1] = "coal-red";
      if (logCoalEdges[col] + 2 <= rowCount - 1) {
        coals[col][logCoalEdges[col] + 2] = "coal-red";
      }
      for (let row = logCoalEdges[col] + 3; row < rowCount; row++) {
        coals[col][row] = "coal-yellow";
      }
    }
  }

  let blocks = Array.from({ length: rowCount }, () =>
    Array(colCount).fill(null)
  );

  let timeoutId = null;
  const calculateNextTableau = () => {
    blocks = Array.from({ length: rowCount }, () => Array(colCount).fill(null));
    // below log action: advance existing embers, place new ones
    const newEmbers: ColorMap = {};
    for (let col in logCoalEdges) {
      if (!newEmbers[col]) {
        newEmbers[col] = {};
      }
      for (let rowKey in embers[col]) {
        const row = +rowKey;
        const currentStage = emberColors.indexOf(embers[col][row]);
        const nextRow = Math.ceil(
          (Math.random() * (rowCount - logFlameEdges[col] - 1)) / 4
        );
        if (currentStage < 2 && row - nextRow > logFlameEdges[col] - 1) {
          newEmbers[col][row - nextRow] = emberColors[currentStage + 1];
        } else if (row - nextRow > logFlameEdges[col] - 1) {
          newEmbers[col][row - nextRow] = emberColors[currentStage];
        }
      }
      let numEmbers = Object.keys(newEmbers[col]).length;
      const idealEmbers = Math.floor((rowCount - logCoalEdges[col]) / 3);
      if (numEmbers < idealEmbers) {
        if (
          Math.random() < 0.75 &&
          Math.max(...Object.keys(newEmbers[col]).map(Number)) < rowCount - 2
        ) {
          newEmbers[col][rowCount - 1] = "ember1";
        }
      }
    }

    const flames: ColorMap = {};
    for (let col in logFlameEdges) {
      flames[col] = {};
      const height = Math.min(
        logFlameEdges[col],
        3 * (rowCount - logFlameEdges[col])
      );
      const heightChunk = Math.floor(height / 3);
      // for the third above the log, 75% likelihood
      for (let i = 1; i <= heightChunk; i++) {
        if (Math.random() < 0.75) {
          flames[col][logFlameEdges[col] - i] = "flame-red";
        }
      }
      // for the third above, 50% likelihood
      for (let i = 1; i <= heightChunk; i++) {
        if (Math.random() < 0.5) {
          flames[col][logFlameEdges[col] - i - heightChunk] = "flame-red";
        }
      }
      // for the final third, 25% likelihood
      for (let i = 1; i <= heightChunk; i++) {
        if (Math.random() < 0.25) {
          flames[col][logFlameEdges[col] - i - heightChunk * 2] = "flame-red";
        }
      }
    }
    for (let col in flames) {
      for (let row in flames[col]) {
        let neighbors = 0;
        if (flames[+col - 2]?.[+row - 2]) {
          neighbors++;
        }
        if (flames[+col + 2]?.[+row - 2]) {
          neighbors++;
        }
        if (flames[+col - 2]?.[+row + 2]) {
          neighbors++;
        }
        if (flames[+col + 2]?.[+row + 2]) {
          neighbors++;
        }
        if (neighbors === 2) {
          flames[col][row] = "flame-orange";
        } else if (neighbors > 2) {
          flames[col][row] = "flame-yellow";
        }
      }
    }

    for (let col in logs) {
      for (let row in logs[col]) {
        blocks[row][col] = logs[col][row];
      }
    }
    for (let col in coals) {
      for (let row in coals[col]) {
        blocks[row][col] = coals[col][row];
      }
    }
    embers = newEmbers;
    for (let col in embers) {
      for (let row in embers[col]) {
        blocks[row][col] = embers[col][row];
      }
    }
    for (let col in flames) {
      for (let row in flames[col]) {
        blocks[row][col] = flames[col][row];
      }
    }
    blocks = blocks;
  };
  const scheduleNextTableau = () => {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => {
      calculateNextTableau();
      scheduleNextTableau();
    }, 250);
  };
  scheduleNextTableau();
</script>

<div class="fireplace-container">
  {#each blocks as blockRow}
    <div class="fireplace-row">
      {#each blockRow as block}
        <Block color={block} />
      {/each}
    </div>
  {/each}
</div>

<style>
  .fireplace-container {
    width: 1024px;
    height: 512px;
    box-sizing: border-box;
  }

  .fireplace-row {
    display: flex;
  }
</style>
