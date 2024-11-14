<script>
  import svelteLogo from './assets/svelte.svg';
  import viteLogo from '/vite.svg';
  import { writable} from 'svelte/store'
  

  let rows = 26;
  let cols = 26;
  let letters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';

  let selectedCell = $state([]);
  let editedCell = $state('');

  let data = $state(Array.from({ length: rows }, () => Array(cols).fill({value: ''})));

  const parseFormula = (data) => {
    if (data.includes("=")) {
      let [s1,s2] = data.split('(')
      let action = s1.split('=')[1]
      let cells = s2.replace(')',' ').split(',')
      let cellIndexs = cells.map(cellToIndex)

      let res = getReduced(cellIndexs,action.toLowerCase())
      let [selectedRow,selectedCol] = selectedCell;

      // data update
      return res
    }

    return data
  }

  const cellToIndex = (cell) => {
    const [col,...row] = cell.split('');
    return {
      row: +row.join('') - 1,
      col: letters.indexOf(col)
    }
  }

  const getReduced = (cellIndexObjs,action) => {
    
    let values = []
    for(let cell of cellIndexObjs) {
      // console.log(+data[cell.row][cell.col].value)
      values.push(+data[cell.row]?.[cell.col]?.value)
    }

    return values.reduce((acc,val) => {
      if (action == "add") return acc + val;
      if (action == "multiply") return acc * val;
    }, action == "add" ? 0 : 1 )

  }

  const update = (e) => {
    const { value, parentElement } = e.target;
    const { row , col} = cellToIndex(parentElement.dataset.cell)

    if (data[row]) {
      if (data[row][col]) {
        data[row][col].value = value;
      } else {
        data[row][col] = { value }
      }
    } else {
      data[row] = []
      data[row][col] = { value }
    }
  }

  $inspect(selectedCell).with(console.log)

</script>

<main>

  <table>
    <thead>
      <tr>
        <th></th>
        {#each Array(rows) as _ , i}
          <th>{letters[i]}</th>
        {/each}
      </tr>      
    </thead>
    <tbody>
      {#each Array(rows) as _, i}
        <tr>
          <th>{i + 1}</th>
          {#each Array(cols) as _ , k}
            {@const cell_num = `${letters[k]}${i+1}`}
            {@const value = data[i]?.[k]?.value}
            {@const parsedValue = value && parseFormula(value)}
            <td data-cell={cell_num} onclick={() => selectedCell = [i,k]} ondblclick={() => editedCell = cell_num}>
              {#if editedCell === cell_num}
                <input type="text" {value}
                  oninput={update} autofocus
                />
              {:else}
                <span>{parsedValue}</span>
              {/if}
            </td>
          {/each}
        </tr>
      {/each}
    </tbody>
  </table>

</main>

<style>
  * {
    padding: 0;
    margin: 0;
    box-sizing: border-box;
  }

  main{      
    width: 100vw;
    height: 100vh;
  }

  table {
    border-collapse: separate;
    border-spacing: 3px;
    padding: 4px;
    border: 1px solid gray;
    width: 100%;
    height: 100%;
  }

  table td {
    position:relative;
    border: 1px solid black;
    width: 100px;
    overflow:hidden;
    cursor: col-resize;
    resize: horizontal;
  }

  table td > input {
    width: 100%;
    height: 20px;
    outline: none;
    border: none;
    font-family: -apple-system, BlinkMacSystemFont, sans-serif;
    font-weight: 700;
    font-size: 15px;
  }

  table td:focus-within {
    border: 1px solid blue;
  }

</style>
<!-- onchange={() => parseFormula(data[i][k].value)} -->