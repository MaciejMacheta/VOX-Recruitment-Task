<template>
  <div class="container">
    <button class="tileAddButton" @click="addTile">Dodaj kafelek</button>
    <div class="coloredDivContainer">
      <div class="colored-grid">
        <transition-group name="fade">
          <div
            v-for="(tile, index) in tiles"
            :key="index"
            :style="{ backgroundColor: tile.bgColor }"
            class="colored-div"
            :class="{ 'start-new-column': index % 4 === 0 && index !== 0, 'show': tile.show, 'dragging': tile.dragging }"
            draggable="true"
            @dragstart="dragStart($event, index)"
            @dragover="dragOver($event)"
            @dragend="dragEnd"
            @drop="drop($event, index)"
          >
            <button @click="openColorPicker(index)">Zmień</button>
            <button @click="removeTile(index)">Usuń</button>
          </div>
        </transition-group>
      </div>
    </div>
    <ColorPickerModal
    v-if="selectedTile && selectedTile.showColorPicker"
    :tile="selectedTile"
    @changeColor="changeTileColor"
    @showColorPicker="handleShowColorPicker"
  />
  </div>
</template>


<script>
import { ref } from "vue";

import { getRandomColor } from "./components/RandomColors";
import ColorPickerModal from "./components/ColorPickerModal.vue";

export default {
  components: {
    ColorPickerModal,
  },
  setup() {
    const tiles = ref([]);
    const selectedTile = ref(null);
    const newTileRef = ref(null); 
    const draggingTileIndex = ref(null); 

    const addTile = () => {
      if (tiles.value.length < 28) {
        const newTile = {
          bgColor: getRandomColor(),
          rgbColor: "",
          rgb: { r: 0, g: 0, b: 0, a: 1 },
          colorFormat: "hex",
          showColorPicker: false,
          show: false,
        };

        tiles.value.push(newTile);
        newTileRef.value = newTile;

        setTimeout(() => {
          newTileRef.value.show = true;
        }, 50); 
      } else {
        alert("Osiągnięto maksymalną liczbę kafelków");
      }
    };


    const dragStart = (event, index) => {
  event.dataTransfer.setData("index", index.toString());
  draggingTileIndex.value = index;
  tiles.value[index].dragging = true; 
};

const dragEnd = () => {
  draggingTileIndex.value = null;
  tiles.value.forEach((tile) => {
    tile.dragging = false; 
  });
};

    const dragOver = (event) => {
      event.preventDefault();
    };

    const drop = (event, targetIndex) => {
      event.preventDefault();
      const draggedIndex = parseInt(event.dataTransfer.getData("index"));
      const temp = tiles.value[targetIndex];
      tiles.value[targetIndex] = tiles.value[draggedIndex];
      tiles.value[draggedIndex] = temp;
    };

    const handleShowColorPicker = (value) => {
      tiles.value.forEach((tile) => {
        tile.showColorPicker = value;
      });
    };

    const removeTile = (index) => {
      tiles.value.splice(index, 1);
    };

    const changeTileColor = (changedTile) => {
      const index = tiles.value.findIndex(tile => tile === selectedTile.value);
      if (index !== -1) {
        tiles.value[index] = { ...changedTile };
      }
      selectedTile.value.showColorPicker = false;
    };

    
    const openColorPicker = (index) => {
  selectedTile.value = tiles.value[index];
  selectedTile.value.showColorPicker = true; 
  tiles.value.forEach((tile, i) => {
    tile.showColorPicker = i === index;
  });
};


    return {
      tiles,
      selectedTile,
      addTile,
      dragStart,
      dragOver,
      dragEnd,
      drop,
      openColorPicker,
      handleShowColorPicker,
      removeTile,
      changeTileColor,
      openColorPicker,
      draggingTileIndex,
    };
  },
};
</script>

