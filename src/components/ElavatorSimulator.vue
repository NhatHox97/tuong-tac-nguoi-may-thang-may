<template>
    <div class="elevator-simulator">
        <div class="handle">
            <div class="display">Lorem ipsum dolor sit amet, consecteteuer adpsim elit</div>
            <div class="buttons">
                <div class="btn-container bottom">
                    <!-- Set floor T to 0 -->
                    <button class="btn-floor" data-set-floor="0">T</button>
                </div>
                <div v-for="i in 6" :key="i" class="btn-container floor">
                    <button class="btn-floor" :data-set-floor="i">{{ i }}</button>
                </div>
            </div>
        </div>
        <div class="building">
            <div class="elevator-container">
                <div class="elevator">
                    <div class="elevator-door"></div>
                    <div class="elevator-light"></div>
                </div>
            </div>
            <div class="floors">
                <!-- Adjust the loop to match the button-floor mapping -->
                <div v-for="i in 7" :key="i" class="floor" :data-floor="i - 1">
                    <div v-if="i - 1 === 0" class="floor-door"></div>
                    <div v-else class="floor-window btn-container bottom">
                        <!-- Up arrow -->
                         <button class="arrow" :data-set-floor="i - 1">UP</button>
                         <button class="arrow" :data-set-floor="i - 1">DOWN</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'ElevatorComponent',
    mounted() {
        this.initializeElevator();
    },
    methods: {
        initializeElevator() {
            let elevator = this.$el.querySelector(".elevator");
            let elevatorDoor = elevator.querySelector(".elevator-door");
            // let elevatorLight = elevator.querySelector(".elevator-light");
            let floors = this.$el.querySelectorAll(".building .floor");
            let buttons = this.$el.querySelectorAll(".handle button");
            let arrows = this.$el.querySelectorAll(".arrow");
            let display = this.$el.querySelector(".display");

            var destinyFloors = [];
            var currentFloor = null;
            var leavingFloor = false;
            var elevatorStatus = 'idle';
            var elevatorWaitingTime = 500;
            var elevatorWaitTime = 500;
            var previousTime = new Date().getTime();
            var deltaTime = 0;

            elevatorDoor.style.width = "1px";
            elevator.style.top = floors[0].offsetTop + "px";

            buttons.forEach(button => {
                button.addEventListener("click", function () {
                    console.log("=============== listen here")
                    let setFloor = this.getAttribute("data-set-floor");
                    let selectedFloor = Array.from(floors).find(f => f.getAttribute("data-floor") == setFloor);

                    if (!destinyFloors.some(df => df.getAttribute("data-floor") == selectedFloor.getAttribute("data-floor"))) {
                        if (selectedFloor.getAttribute("data-floor") != currentFloor?.getAttribute("data-floor")) {
                            destinyFloors.push(selectedFloor);
                        }
                    }
                    leavingFloor = true;
                    if (elevatorStatus === 'idle') {
                        elevatorStatus = 'closing';
                    }
                });
            });


            arrows.forEach(button => {
                button.addEventListener("click", function () {
                    console.log("=============== arrow")
                    console.log("==============", this)
                    let setFloor = this.getAttribute("data-set-floor");
                    let selectedFloor = Array.from(floors).find(f => f.getAttribute("data-floor") == setFloor);

                    if (!destinyFloors.some(df => df.getAttribute("data-floor") == selectedFloor.getAttribute("data-floor"))) {
                        if (selectedFloor.getAttribute("data-floor") != currentFloor?.getAttribute("data-floor")) {
                            destinyFloors.push(selectedFloor);
                        }
                    }
                    leavingFloor = true;
                    if (elevatorStatus === 'idle') {
                        elevatorStatus = 'closing';
                    }
                });
            });

            const updateElevator = () => {
                deltaTime = new Date().getTime() - previousTime;
                previousTime = new Date().getTime();

                requestAnimationFrame(updateElevator);
                var elevatorWithinFloor = false;
                for (let i = 0; i < floors.length; i++) {
                    if (elevator.offsetTop > floors[i].offsetTop && elevator.offsetTop < floors[i].offsetTop + 10) {
                        elevatorWithinFloor = true;
                        currentFloor = floors[i];

                        if (!leavingFloor) {
                            if (destinyFloors.some(df => df.getAttribute("data-floor") == currentFloor.getAttribute("data-floor"))) {
                                destinyFloors = destinyFloors.filter(df => df.getAttribute("data-floor") != currentFloor.getAttribute("data-floor"));
                                elevatorStatus = 'opening';
                            }
                        }
                    }
                }

                if (!elevatorWithinFloor) {
                    if (leavingFloor) {
                        leavingFloor = false;
                    }
                }

                if (elevatorStatus !== 'moving') {
                    if (elevatorStatus === 'opening') {
                        if (elevatorDoor.offsetWidth > 1) {
                            elevatorDoor.style.width = (elevatorDoor.offsetWidth - 1) + "px";
                        } else {
                            if (destinyFloors.length === 0) {
                                elevatorStatus = 'idle';
                            } else {
                                elevatorStatus = 'waiting';
                                elevatorWaitingTime = elevatorWaitTime;
                            }
                        }
                    }
                    if (elevatorStatus === 'waiting') {
                        if (elevatorWaitingTime > 0) {
                            elevatorWaitingTime -= deltaTime;
                        } else {
                            elevatorStatus = 'closing';
                        }
                    }
                    if (elevatorStatus === 'closing') {
                        if (elevatorDoor.offsetWidth < 34) {
                            elevatorDoor.style.width = (elevatorDoor.offsetWidth + 1) + "px";
                        } else {
                            elevatorStatus = 'moving';
                        }
                    }
                }

                if (destinyFloors[0] != null && elevatorStatus === 'moving') {
                    if (destinyFloors[0].offsetTop > elevator.offsetTop - 7) {
                        elevator.style.top = (elevator.offsetTop - 7 + 2) + "px";
                    } else {
                        elevator.style.top = (elevator.offsetTop - 7 - 2) + "px";
                    }
                }

                this.updateButtons(buttons, destinyFloors);
                this.updateDisplay(display, currentFloor, destinyFloors);
            };
            updateElevator();
        },

        updateDisplay(display, currentFloor, destinyFloors) {
            const floorNames = [
                "Tầng trệt",
                "Tầng 1",
                "Tầng 2",
                "Tầng 3",
                "Tầng 4",
                "Tầng 5",
                "Tầng 6",
            ];

            display.innerHTML = `${floorNames[parseInt(currentFloor.getAttribute("data-floor"))]} ${destinyFloors[0] != null ? (destinyFloors[0].offsetTop < currentFloor.offsetTop ? '<br />Đi lên' : '<br />Đi xuống') : ''
                }`;
        },

        updateButtons(buttons, destinyFloors) {
            buttons.forEach(button => {
                if (destinyFloors.some(df => df.getAttribute("data-floor") === button.getAttribute("data-set-floor"))) {
                    button.classList.add("active");
                } else {
                    button.classList.remove("active");
                }
            });
        }
    }
};
</script>

<style lang="scss" scoped>
@import '@/assets/styles/ElevatorStyles.scss';


.arrow {
    position: relative;
    left: 100%
}
.arrow {
    position: relative;
    left: 100%
}
</style>