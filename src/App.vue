<template>
    <div>
        <h1>Data Visualization</h1>

        <!-- 파일 선택 드롭다운 -->
        <label for="file-select">Select Data File: </label>
        <select id="file-select" v-model="selectedDataFile" @change="loadCSV">
            <option v-for="fileName in dataFileNames" :key="fileName" :value="fileName">
                {{ fileName }}
            </option>
        </select>

        <div v-html="displayData"></div>
        <!-- 항상 표시될 데이터 위치 -->

        <Splitter style="height: 1200px" layout="vertical">
            <!-- <SplitterPanel class="flex" :size="31">
                <div ref="candleChartContainer" style="width: 100%; height: 300px" class="firstchart">Candle Chart</div>
            </SplitterPanel> -->
            <SplitterPanel class="flex" :size="31">
                <ScrollPanel style="width: 80%; height: 850px">
                    <div style="width: 95%">
                        <div ref="candleChartContainer" style="width: 100%; height: 300px" class="firstchart">Candle Chart</div>
                    </div>
                </ScrollPanel>
                <ScrollPanel style="width: 20%" class="p-2">
                    <div class="flex gap-2">
                        <Button @click="runVisual()">적용</Button>
                        <Button @click="deletevalue()">초기화</Button>
                    </div>
                    <div class="card flex gap-3 p-2">
                        <div>ADX1 <Checkbox v-model="conditionADX1" :binary="true" /></div>
                        <div>ADX2 <Checkbox v-model="conditionADX2" :binary="true" /></div>
                        <div>MACD1 <Checkbox v-model="conditionMACD1" :binary="true" /></div>
                        <div>MACD2 <Checkbox v-model="conditionMACD2" :binary="true" /></div>
                        <div>ST <Checkbox v-model="conditionST" :binary="true" /></div>
                        <div>RSI <Checkbox v-model="conditionRSI" :binary="true" /></div>
                        <div>WT1 <Checkbox v-model="conditionWT1" :binary="true" /></div>
                        <div>WT2 <Checkbox v-model="conditionWT2" :binary="true" /></div>
                    </div>
                    <div v-if="conditionADX1" class="card flex p-2">
                        <div>ADX1</div>
                        <OrderList v-model="adxindicatorsafter" :dragdrop="true" header="Indicator Order" v-show="conditionADX1">
                            <template #header>바뀐자료</template>
                            <template #item="slotProps">
                                <div class="p-d-flex p-ai-center p-jc-between" style="width: 100%">
                                    <div :style="{ color: getColor(slotProps.item.name) }">{{ slotProps.index + 1 }}. {{ slotProps.item.name }}</div>
                                </div>
                            </template>
                        </OrderList>
                        <div class="ml-4" v-show="conditionADX1">
                            <div>전체자료</div>
                            <Checkbox v-model="adxindicatorsAll" :binary="true" />
                        </div>
                    </div>
                    <div v-if="conditionADX2" class="card flex-row p-2">
                        <div>ADX2</div>
                        <div style="color: red">ADX <InputNumber v-model="adxOver" :disabled="!conditionADX2" />이상일때</div>
                        <div style="color: red">ADX <InputNumber v-model="adxUnder" :disabled="!conditionADX2" />이하일때</div>
                        <div style="color: blue">PDI <InputNumber v-model="pdiOver" :disabled="!conditionADX2" />이상일때</div>
                        <div style="color: blue">PDI <InputNumber v-model="pdiUnder" :disabled="!conditionADX2" />이하일때</div>
                        <div style="color: orange">MDI <InputNumber v-model="mdiOver" :disabled="!conditionADX2" />이상일때</div>
                        <div style="color: orange">MDI <InputNumber v-model="mdiUnder" :disabled="!conditionADX2" />이하일때</div>
                    </div>
                    <div v-if="conditionST" class="card flex p-2">
                        <div>
                            <div>ST</div>
                            <div class="flex align-content-center mb-2">
                                <div class="mr-2">SuperTrend Long</div>
                                <Checkbox v-model="supertrendLong" :binary="true" :disabled="!conditionST"></Checkbox>
                            </div>
                            <div class="flex align-content-center">
                                <div class="mr-2">SuperTrend Short</div>
                                <Checkbox v-model="supertrendShort" :binary="true" :disabled="!conditionST"></Checkbox>
                            </div>
                        </div>
                        <div class="ml-4">
                            <div>전체자료</div>
                            <Checkbox v-model="supertrendAll" :binary="true" :disabled="!conditionST" />
                        </div>
                    </div>
                    <div v-if="conditionWT1" class="card flex p-2">
                        <div>
                            <div>WaveTrend1</div>
                            <div class="flex align-content-center mb-2">
                                <div class="mr-2">WaveTrend Long</div>
                                <Checkbox v-model="wavetrendLong" :binary="true" :disabled="!conditionWT1"></Checkbox>
                            </div>
                            <div class="flex align-content-center">
                                <div class="mr-2">WaveTrend Short</div>
                                <Checkbox v-model="wavetrendShort" :binary="true" :disabled="!conditionWT1"></Checkbox>
                            </div>
                        </div>
                        <div class="ml-4">
                            <div>전체자료</div>
                            <Checkbox v-model="wavetrendAll" :binary="true" :disabled="!conditionWT1" />
                        </div>
                    </div>
                    <div v-if="conditionWT2" class="card flex p-2">
                        <div>WaveTrend2</div>
                        <div style="color: blue">WT1<InputNumber v-model="wt1Over" :disabled="!conditionWT2" /> 이상일때</div>
                        <div style="color: blue">WT1<InputNumber v-model="wt1Under" :disabled="!conditionWT2" /> 이하일때</div>
                        <div style="color: red">WT2<InputNumber v-model="wt2Over" :disabled="!conditionWT2" /> 이상일때</div>
                        <div style="color: red">WT2<InputNumber v-model="wt2Under" :disabled="!conditionWT2" /> 이하일때</div>
                    </div>

                    <div v-if="conditionRSI" class="card flex-row p-2">
                        <div>RSI</div>
                        <div><InputNumber v-model="rsiOver" :disabled="!conditionRSI" /> 이상일때</div>
                        <div><InputNumber v-model="rsiUnder" :disabled="!conditionRSI" /> 이하일때</div>
                    </div>
                    <div v-if="conditionMACD1" class="card flex-row p-2">
                        <div>MACD1</div>
                        <div style="color: blue">MACD<InputNumber v-model="macdOver" :disabled="!conditionMACD1" />이상일때</div>
                        <div style="color: blue">MACD<InputNumber v-model="macdUnder" :disabled="!conditionMACD1" />이하일때</div>
                        <div style="color: red">Signal<InputNumber v-model="signalOver" :disabled="!conditionMACD1" />이상일때</div>
                        <div style="color: red">Signal<InputNumber v-model="signalUnder" :disabled="!conditionMACD1" />이하일때</div>
                    </div>
                    <div v-if="conditionMACD2" class="card flex p-2">
                        <div>
                            <div>MACD2</div>
                            <div class="mt-2">
                                <div class="flex align-content-center mb-2">
                                    <div style="color: blue" class="mr-2">MACD가Signal보다 클때</div>
                                    <div><Checkbox v-model="macdMore" :binary="true" :disabled="!conditionMACD2"></Checkbox></div>
                                </div>
                                <div class="flex align-content-center mb-2">
                                    <div style="color: red" class="mr-2">Signal가MACD보다 클때</div>
                                    <div><Checkbox v-model="signalMore" :binary="true" :disabled="!conditionMACD2"></Checkbox></div>
                                </div>
                            </div>
                        </div>
                        <div class="ml-4">
                            <div>전체자료</div>
                            <Checkbox v-model="macdAll" :binary="true" :disabled="!conditionMACD2" />
                        </div>
                    </div>
                </ScrollPanel>
            </SplitterPanel>
            <SplitterPanel class="flex" :size="69">
                <ScrollPanel style="width: 80%; height: 850px">
                    <div style="width: 95%" class="chart-container">
                        <div ref="volumeChartContainer" style="width: 100%; height: 100px; margin-bottom: 20px" :class="{ 'top-chart': topChart === 'volume' }">Volume Chart</div>
                        <div ref="adxChartContainer" style="width: 100%; height: 200px; margin-bottom: 20px" :class="{ 'top-chart': topChart === 'adx' }">ADX Chart</div>
                        <div ref="supertrendChartContainer" style="width: 100%; height: 200px; margin-bottom: 20px" :class="{ 'top-chart': topChart === 'supertrend' }">Supertrend Chart</div>
                        <div ref="wavetrendChartContainer" style="width: 100%; height: 200px; margin-bottom: 20px" :class="{ 'top-chart': topChart === 'wavetrend' }">WaveTrend Chart</div>
                        <div ref="rsiChartContainer" style="width: 100%; height: 200px; margin-bottom: 20px" :class="{ 'top-chart': topChart === 'rsi' }">RSI Chart</div>
                        <div ref="macdChartContainer" style="width: 100%; height: 200px; margin-bottom: 120px" :class="{ 'top-chart': topChart === 'macd' }">MACD Chart</div>
                    </div>
                </ScrollPanel>
                <ScrollPanel style="width: 20%; height: 850px">
                    <div class="flex gap-2">
                        <Button @click="moveChartToTop('volume')" class="mb-2">Volume</Button>
                        <Button @click="moveChartToTop('adx')" class="mb-2">ADX</Button>
                        <Button @click="moveChartToTop('supertrend')" class="mb-2">Supertrend</Button>
                        <Button @click="moveChartToTop('wavetrend')" class="mb-2">WaveTrend</Button>
                        <Button @click="moveChartToTop('rsi')" class="mb-2">RSI</Button>
                        <Button @click="moveChartToTop('macd')" class="mb-2">MACD</Button>
                    </div>
                    <div class="card flex p-2">
                        <Listbox v-model="selectFinalList" :options="finalList" optionLabel="label" style="width: 100%" @change="scrollToTime"> </Listbox>
                        <div v-if="selectFinalList">
                            <p>선택된 Time: {{ selectFinalList.value }}</p>
                        </div>
                    </div>
                </ScrollPanel>
            </SplitterPanel>
        </Splitter>
        <!-- <div class="flex">
            <div style="width: 72%" class="chart-container">
                <div ref="candleChartContainer" style="width: 100%; height: 350px; margin-bottom: 20px" class="firstchart">Candle Chart</div>
                <div ref="volumeChartContainer" style="width: 100%; height: 100px; margin-bottom: 20px" :class="{ 'top-chart': topChart === 'volume' }">Volume Chart</div>
                <div ref="adxChartContainer" style="width: 100%; height: 200px; margin-bottom: 20px" :class="{ 'top-chart': topChart === 'adx' }">ADX Chart</div>
                <div ref="supertrendChartContainer" style="width: 100%; height: 200px; margin-bottom: 20px" :class="{ 'top-chart': topChart === 'supertrend' }">Supertrend Chart</div>
                <div ref="wavetrendChartContainer" style="width: 100%; height: 200px; margin-bottom: 20px" :class="{ 'top-chart': topChart === 'wavetrend' }">WaveTrend Chart</div>
                <div ref="rsiChartContainer" style="width: 100%; height: 200px; margin-bottom: 20px" :class="{ 'top-chart': topChart === 'rsi' }">RSI Chart</div>
                <div ref="macdChartContainer" style="width: 100%; height: 200px; margin-bottom: 20px" :class="{ 'top-chart': topChart === 'macd' }">MACD Chart</div>
            </div>
            <div style="width: 8%">
                <div class="button-group">
                    <Button @click="moveChartToTop('volume')" class="mb-2">Volume Chart</Button>
                    <Button @click="moveChartToTop('adx')" class="mb-2">ADX Chart</Button>
                    <Button @click="moveChartToTop('supertrend')" class="mb-2">Supertrend Chart</Button>
                    <Button @click="moveChartToTop('wavetrend')" class="mb-2">WaveTrend Chart</Button>
                    <Button @click="moveChartToTop('rsi')" class="mb-2">RSI Chart</Button>
                    <Button @click="moveChartToTop('macd')" class="mb-2">MACD Chart</Button>
                </div>
            </div>

            <div style="width: 20%">
                <Button @click="runVisual()">적용</Button>
                <Button @click="deletevalue()">초기화</Button>
                <div class="card flex gap-3 p-2">
                    <div>ADX1 <Checkbox v-model="conditionADX1" :binary="true" /></div>
                    <div>ADX2 <Checkbox v-model="conditionADX2" :binary="true" /></div>
                    <div>MACD1 <Checkbox v-model="conditionMACD1" :binary="true" /></div>
                    <div>MACD2 <Checkbox v-model="conditionMACD2" :binary="true" /></div>
                    <div>ST <Checkbox v-model="conditionST" :binary="true" /></div>
                    <div>RSI <Checkbox v-model="conditionRSI" :binary="true" /></div>
                    <div>WT <Checkbox v-model="conditionWT" :binary="true" /></div>
                </div>

                <div class="card flex p-2">
                    <div>ADX1</div>
                    <OrderList v-model="adxindicatorsafter" :dragdrop="true" header="Indicator Order" v-show="conditionADX1">
                        <template #header>바뀐자료</template>
                        <template #item="slotProps">
                            <div class="p-d-flex p-ai-center p-jc-between" style="width: 100%">
                                <div :style="{ color: getColor(slotProps.item.name) }">{{ slotProps.index + 1 }}. {{ slotProps.item.name }}</div>
                            </div>
                        </template>
                    </OrderList>
                    <div class="ml-4" v-show="conditionADX1">
                        <div>전체자료</div>
                        <Checkbox v-model="adxindicatorsAll" :binary="true" />
                    </div>
                </div>

                <div class="card flex-row p-2">
                    <div>ADX2</div>
                    <div style="color: red">ADX <InputNumber v-model="adxOver" :disabled="!conditionADX2" />이상일때</div>
                    <div style="color: red">ADX <InputNumber v-model="adxUnder" :disabled="!conditionADX2" />이하일때</div>
                    <div style="color: blue">PDI <InputNumber v-model="pdiOver" :disabled="!conditionADX2" />이상일때</div>
                    <div style="color: blue">PDI <InputNumber v-model="pdiUnder" :disabled="!conditionADX2" />이하일때</div>
                    <div style="color: orange">MDI <InputNumber v-model="mdiOver" :disabled="!conditionADX2" />이상일때</div>
                    <div style="color: orange">MDI <InputNumber v-model="mdiUnder" :disabled="!conditionADX2" />이하일때</div>
                </div>
                <div class="card flex p-2">
                    <div>
                        <div>ST</div>
                        <div class="flex align-content-center mb-2">
                            <div class="mr-2">SuperTrend Long</div>
                            <Checkbox v-model="supertrendLong" :binary="true" :disabled="!conditionST"></Checkbox>
                        </div>
                        <div class="flex align-content-center">
                            <div class="mr-2">SuperTrend Short</div>
                            <Checkbox v-model="supertrendShort" :binary="true" :disabled="!conditionST"></Checkbox>
                        </div>
                    </div>
                    <div class="ml-4">
                        <div>전체자료</div>
                        <Checkbox v-model="supertrendAll" :binary="true" :disabled="!conditionST" />
                    </div>
                </div>
                <div class="card flex p-2">
                    <div>
                        <div>WT</div>
                        <div class="flex align-content-center mb-2">
                            <div class="mr-2">WaveTrend Long</div>
                            <Checkbox v-model="wavetrendLong" :binary="true" :disabled="!conditionWT"></Checkbox>
                        </div>
                        <div class="flex align-content-center">
                            <div class="mr-2">WaveTrend Short</div>
                            <Checkbox v-model="wavetrendShort" :binary="true" :disabled="!conditionWT"></Checkbox>
                        </div>
                    </div>
                    <div class="ml-4">
                        <div>전체자료</div>
                        <Checkbox v-model="wavetrendAll" :binary="true" :disabled="!conditionWT" />
                    </div>
                </div>
                <div class="card flex-row p-2">
                    <div>RSI</div>
                    <div><InputNumber v-model="rsiOver" :disabled="!conditionRSI" /> 이상일때</div>
                    <div><InputNumber v-model="rsiUnder" :disabled="!conditionRSI" /> 이하일때</div>
                </div>
                <div class="card flex-row p-2">
                    <div>MACD1</div>
                    <div style="color: blue">MACD<InputNumber v-model="macdOver" :disabled="!conditionMACD1" />이상일때</div>
                    <div style="color: blue">MACD<InputNumber v-model="macdUnder" :disabled="!conditionMACD1" />이하일때</div>
                    <div style="color: red">Signal<InputNumber v-model="signalOver" :disabled="!conditionMACD1" />이상일때</div>
                    <div style="color: red">Signal<InputNumber v-model="signalUnder" :disabled="!conditionMACD1" />이하일때</div>
                </div>
                <div class="card flex p-2">
                    <div>
                        <div>MACD2</div>
                        <div class="mt-2">
                            <div class="flex align-content-center mb-2">
                                <div style="color: blue" class="mr-2">MACD가Signal보다 클때</div>
                                <div><Checkbox v-model="macdMore" :binary="true" :disabled="!conditionMACD2"></Checkbox></div>
                            </div>
                            <div class="flex align-content-center mb-2">
                                <div style="color: red" class="mr-2">Signal가MACD보다 클때</div>
                                <div><Checkbox v-model="signalMore" :binary="true" :disabled="!conditionMACD2"></Checkbox></div>
                            </div>
                        </div>
                    </div>
                    <div class="ml-4">
                        <div>전체자료</div>
                        <Checkbox v-model="macdAll" :binary="true" :disabled="!conditionMACD2" />
                    </div>
                </div>
            </div>
        </div> -->
    </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue';
import { createChart } from 'lightweight-charts';
import * as d3 from 'd3';
// 핫키 설정(키보드 단축키 1 누르면 topChart가 volume로 변경)
import hotkeys from 'hotkeys-js';
hotkeys('1', (event, handler) => {
    moveChartToTop('volume');
});
hotkeys('2', (event, handler) => {
    moveChartToTop('adx');
});
hotkeys('3', (event, handler) => {
    moveChartToTop('supertrend');
});
hotkeys('4', (event, handler) => {
    moveChartToTop('wavetrend');
});
hotkeys('5', (event, handler) => {
    moveChartToTop('rsi');
});
hotkeys('6', (event, handler) => {
    moveChartToTop('macd');
});

// 초기화 함수(마커 및 각 수치 초기화)
function deletevalue() {
    // 페이지를 리프레시 함
    location.reload();
}

//조건부 펑션함수
function runVisual() {
    // 조건을 만족하는 인덱스를 저장할 배열들
    const stMarkers = [];
    const wt1Makers = [];
    const wt2Makers = [];

    const rsiMarkers = [];
    const macd1Markers = [];
    const macd2Markers = [];
    const adx1Markers = [];
    const adx2Markers = [];

    data.value.forEach((item, i) => {
        const prevItem = data.value[i - 1];

        // 1. SuperTrend 조건
        if (conditionST.value) {
            // Long 조건: supertrendAll이 true일 경우 이전 항목 비교 없이 모든 lowerBand 항목 추가
            if (supertrendLong.value && item.lowerBand && (supertrendAll.value || !prevItem || !prevItem.lowerBand)) {
                stMarkers.push(i);
            }
            // Short 조건: supertrendAll이 true일 경우 이전 항목 비교 없이 모든 upperBand 항목 추가
            if (supertrendShort.value && item.upperBand && (supertrendAll.value || !prevItem || !prevItem.upperBand)) {
                stMarkers.push(i);
            }
        }
        // 2. wavetrend1 조건
        if (conditionWT1.value) {
            let addMarker = false;

            if (wavetrendAll.value) {
                if ((wavetrendLong.value && item.wt1 > item.wt2) || (wavetrendShort.value && item.wt2 > item.wt1)) {
                    addMarker = true;
                }
            } else {
                // macdAll이 false일 때는 이전 항목과 비교하여 상태가 전환된 시점만 표시
                if (wavetrendLong.value && item.wt1 > item.wt2 && (!prevItem || prevItem.wt1 <= prevItem.wt2)) {
                    addMarker = true;
                }
                if (wavetrendShort.value && item.wt2 > item.wt1 && (!prevItem || prevItem.wt2 <= prevItem.wt1)) {
                    addMarker = true;
                }
            }

            // 조건에 따라 마커를 추가
            if (addMarker) {
                wt1Makers.push(i);
            }
        } else {
            // 조건이 활성화되지 않은 경우 모든 항목을 포함
            wt1Makers.push(i);
        }

        // 1-1 wavetrend 조건2 wt1과 wt2의 숫자 비교
        if (conditionWT2.value) {
            // wt1 조건 평가
            let wt1Condition = true; // 기본값
            let wt2Condition = true; // 기본값
            if (wt1Over?.value !== null && wt1Over?.value !== undefined) {
                wt1Condition = item.wt1 >= wt1Over.value; // wt1이 wt1Over 이상인지 확인
            }
            if (wt1Under?.value !== null && wt1Under?.value !== undefined) {
                wt1Condition = wt1Condition && item.wt1 <= wt1Under.value; // wt1이 wt1Under 이하인지 확인
            }
            if (wt2Over?.value !== null && wt2Over?.value !== undefined) {
                wt2Condition = item.wt2 >= wt2Over.value; // wt2가 wt2Over 이상인지 확인
            }
            if (wt2Under?.value !== null && wt2Under?.value !== undefined) {
                wt2Condition = wt2Condition && item.wt2 <= wt2Under.value; // wt2가 wt2Under 이하인지 확인
            }

            // 나머지 조건에 대해서 나열해

            if (wt1Condition && wt2Condition) {
                wt2Makers.push(i);
            }
        }

        // 2. RSI 조건
        if (conditionRSI.value) {
            let rsiCondition = true;

            // rsiOver 조건 확인
            if (rsiOver.value !== null && rsiOver.value !== undefined) {
                rsiCondition = item.rsi >= rsiOver.value;
            }

            // rsiUnder 조건 확인
            if (rsiUnder.value !== null && rsiUnder.value !== undefined) {
                rsiCondition = rsiCondition && item.rsi <= rsiUnder.value;
            }

            // 조건을 만족하는 경우 인덱스 추가
            if (rsiCondition) {
                rsiMarkers.push(i);
            }
        }

        /// 3. MACD1 조건
        if (conditionMACD1.value) {
            // MACD 조건 체크 (해당 조건이 활성화된 경우에만 비교)
            const macdConditionMet = (!macdOver.value || item.macd >= macdOver.value) && (!macdUnder.value || item.macd <= macdUnder.value);

            // Signal 조건 체크 (해당 조건이 활성화된 경우에만 비교)
            const signalConditionMet = (!signalOver.value || item.macdSignal >= signalOver.value) && (!signalUnder.value || item.macdSignal <= signalUnder.value);

            // 모든 활성화된 조건이 만족되면 마커 추가
            if (macdConditionMet && signalConditionMet) {
                macd1Markers.push(i);
            }
        } else {
            // 조건이 활성화되지 않은 경우 모든 항목을 포함
            macd1Markers.push(i);
        }

        // 4. MACD2 조건
        if (conditionMACD2.value) {
            let addMarker = false;

            if (macdAll.value) {
                // macdAll이 true일 때는 현재 상태가 macdMore 또는 signalMore 조건을 만족하는지 확인
                if ((macdMore.value && item.macd > item.macdSignal) || (signalMore.value && item.macdSignal > item.macd)) {
                    addMarker = true;
                }
            } else {
                // macdAll이 false일 때는 이전 항목과 비교하여 상태가 전환된 시점만 표시
                if (macdMore.value && item.macd > item.macdSignal && (!prevItem || prevItem.macd <= prevItem.macdSignal)) {
                    addMarker = true;
                }
                if (signalMore.value && item.macdSignal > item.macd && (!prevItem || prevItem.macdSignal <= prevItem.macd)) {
                    addMarker = true;
                }
            }

            // 조건에 따라 마커를 추가
            if (addMarker) {
                macd2Markers.push(i);
            }
        } else {
            // 조건이 활성화되지 않은 경우 모든 항목을 포함
            macd2Markers.push(i);
        }

        // 5. ADX1 조건 (순서 체크)
        if (conditionADX1.value) {
            let isOrderCorrect = true;
            let wasOrderDifferentBefore = false;

            // 현재 항목과 이전 항목이 있는 경우에만 비교
            if (i > 0) {
                for (let j = 0; j < adxindicatorsafter.value.length - 1; j++) {
                    const currentIndicator = adxindicatorsafter.value[j].name.toLowerCase();
                    const nextIndicator = adxindicatorsafter.value[j + 1].name.toLowerCase();

                    // 현재 지표 순서대로 값 비교
                    if (item[currentIndicator] <= item[nextIndicator]) {
                        isOrderCorrect = false;
                        break;
                    }

                    // 이전 값과의 순서가 다른지 확인
                    if (
                        !adxindicatorsAll.value && // 전체 조건을 활성화하지 않은 경우에만
                        prevItem && // 이전 항목이 존재하고
                        prevItem[currentIndicator] <= prevItem[nextIndicator] // 이전 값이 순서가 다른 경우
                    ) {
                        wasOrderDifferentBefore = true;
                    }
                }
            } else {
                // 첫 번째 항목은 순서를 확인할 수 없으므로 무시
                isOrderCorrect = false;
            }

            // 현재 순서가 올바르고, 이전에는 순서가 달랐거나 전체 조건이 활성화된 경우에만 마커 추가
            if (isOrderCorrect && (wasOrderDifferentBefore || adxindicatorsAll.value)) {
                adx1Markers.push(i);
            }
        }

        // 6. ADX2 조건
        if (conditionADX2.value) {
            // ADX 조건 체크 (해당 조건이 활성화된 경우에만 비교)
            const adxConditionMet = (!adxOver.value || item.adx >= adxOver.value) && (!adxUnder.value || item.adx <= adxUnder.value);

            // PDI 조건 체크 (해당 조건이 활성화된 경우에만 비교)
            const pdiConditionMet = (!pdiOver.value || item.pdi >= pdiOver.value) && (!pdiUnder.value || item.pdi <= pdiUnder.value);

            // MDI 조건 체크 (해당 조건이 활성화된 경우에만 비교)
            const mdiConditionMet = (!mdiOver.value || item.mdi >= mdiOver.value) && (!mdiUnder.value || item.mdi <= mdiUnder.value);

            // 모든 활성화된 조건이 만족되면 마커 추가
            if (adxConditionMet && pdiConditionMet && mdiConditionMet) {
                adx2Markers.push(i);
            }
        } else {
            // 조건이 활성화되지 않은 경우 모든 항목을 포함
            adx2Markers.push(i);
        }
    });

    const activeMarkers = [stMarkers, wt1Makers, wt2Makers, rsiMarkers, macd1Markers, macd2Markers, adx1Markers, adx2Markers];
    const activeConditions = [conditionST.value, conditionRSI.value, conditionMACD1.value, conditionMACD2.value, conditionADX1.value, conditionADX2.value];

    const calculateFinalMarkers = (activeMarkers) => {
        // 빈 배열이 아닌 마커 리스트만 필터링
        const nonEmptyMarkers = activeMarkers.filter((markers) => markers.length > 0);

        if (nonEmptyMarkers.length === 0) {
            // 모든 조건이 비어 있는 경우 빈 배열 반환
            return [];
        }

        if (nonEmptyMarkers.length === 1) {
            // 하나의 조건만 활성화된 경우 해당 마커 리스트 반환
            return nonEmptyMarkers[0];
        }

        // 두 개 이상의 조건이 활성화된 경우 교집합 계산
        return nonEmptyMarkers.reduce((acc, markers) => {
            return acc.filter((index) => markers.includes(index));
        });
    };
    //

    // 최종 마커 계산
    const filteredMarkers = calculateFinalMarkers(activeMarkers);

    // 최종 마커 설정
    const markers = filteredMarkers.map((i) => ({
        time: data.value[i].time,
        position: 'aboveBar',
        color: 'blue',
        shape: 'arrowDown'
        //text: '조건 만족'
    }));

    // console.log('stMarkers:', stMarkers);
    // console.log('wt1Makers:', wt1Makers);
    // console.log('rsiMarkers:', rsiMarkers);
    // console.log('macd1Markers:', macd1Markers);
    // console.log('macd2Markers:', macd2Markers);
    // console.log('adx1Markers:', adx1Markers);
    // console.log('adx2Markers:', adx2Markers);

    // console.log('Filtered Markers:', filteredMarkers);
    console.log('Final Markers:', markers);
    updateMarkers(markers);

    candlestickSeries.setMarkers(markers);
}

const finalList = ref([]); // Listbox에 표시될 데이터
const selectFinalList = ref(null); // 선택된 항목
// `markers` 데이터를 처리하고 `finalList`에 저장하는 함수
function updateMarkers(markers) {
    // markers를 finalList 형식으로 변환
    const formattedMarkers = markers.map((marker) => ({
        label: `Time: ${new Date(marker.time * 1000).toLocaleString()}`,
        value: marker.time
    }));

    finalList.value = formattedMarkers; // Listbox에 표시될 데이터 갱신
}
// 캔들 차트를 특정 시간으로 이동시키는 함수
function scrollToTime() {
    if (!selectFinalList.value || !candleChart) {
        console.error('No time selected or chart not available.');
        return;
    }

    const timeScale = candleChart.timeScale();
    const selectedTime = selectFinalList.value; // Unix 타임스탬프 (초 단위)
    console.log('Selected Time:', selectedTime);
    console.log('Selected Time:', selectedTime.value);

    // `timeScale.setVisibleRange`를 사용하여 선택한 시간을 중심으로 범위 설정
    const visibleRangeWidth = 60 * 60 * 24 * 30; // 1시간 범위 (초 단위)
    timeScale.setVisibleRange({
        from: selectedTime.value - visibleRangeWidth / 2,
        to: selectedTime.value + visibleRangeWidth / 2
    });
}

// 현재 최상위 차트를 저장하는 변수
const topChart = ref('volume');

// 특정 차트를 최상위로 이동하는 함수
const moveChartToTop = (chart) => {
    console.log('chart:', chart);
    topChart.value = chart;
};
// 조건관련 변수
const conditionST = ref(false); // supertrend 조건을 넣을지 안 넣을지 (supertrendLong, supertrendShort)
const conditionWT1 = ref(false); // wavetrend 조건을 넣을지 안 넣을지 (WaveTrendLong, WaveTrendShort)
const conditionWT2 = ref(false); // wavetrend 음수양수 조건을 넣을지 안 넣을지 (wavetrendAll)
const conditionRSI = ref(false); // rsi조건을 넣을지 안 넣을지 (rsiOver, rsiUnder)
const conditionMACD1 = ref(false); // macd1조건을 넣을지 안 넣을지 (macdOver, macdUnder, signalOver, signalUnder)
const conditionMACD2 = ref(false); // macd2조건을 넣을지 안 넣을지 (macdMore, signalMore)
const conditionADX1 = ref(false); // adx1 조건을 넣을지 안 넣을지 (adxindicatorsafter)
const conditionADX2 = ref(false); // adx2 조건을 넣을지 안 넣을지 (adxOver, adxUnder, pdiOver, pdiUnder, mdiOver, mdiUnder)

const supertrendLong = ref(false); // supertrend가 long일때
const supertrendShort = ref(false); // supertrend가 short일때
const supertrendAll = ref(false); // supertrend가 long일때
const wavetrendLong = ref(false); // wavetrend가 long일때
const wavetrendShort = ref(false); // wavetrend가 short일때
const wavetrendAll = ref(false); // wavetrend가 long일때

const rsiOver = ref(); // rsi가 이 값 이상일 때
const rsiUnder = ref(); // rsi가 이 값 이하일 때
const macdOver = ref(); // macd가 이 값 이상일 때
const macdUnder = ref(); // macd가 이 값 이하일 때
const signalUnder = ref(); // signal이 이 값 이하일 때
const signalOver = ref(); // signal이 이 값 이상일 때
const macdMore = ref(false); // macd가 signal보다 클때
const signalMore = ref(false); // signal이 macd보다 클때
const macdAll = ref(false); //

const adxOver = ref(); //
const pdiOver = ref();
const mdiOver = ref();
const adxUnder = ref();
const pdiUnder = ref();
const mdiUnder = ref();
const adxindicatorsafter = ref([{ name: 'adx' }, { name: 'pdi' }, { name: 'mdi' }]);
const adxindicatorsAll = ref(false);
const wt1Over = ref();
const wt2Over = ref();
const wt1Under = ref();
const wt2Under = ref();

function getColor(name) {
    switch (name) {
        case 'adx':
            return 'red'; // 예: 빨간색
        case 'pdi':
            return 'blue'; // 예: 파란색
        case 'mdi':
            return 'orange'; // 예: 오랜지색
        default:
            return '#000000'; // 기본 검은색
    }
}
// 데이터 관련 변수
const data = ref([]);
const volumeData = ref([]);
const supertrendData = ref([]);
const wavetrendData = ref([]);
const rsiData = ref([]);
const macdData = ref([]);
const adxData = ref([]);
const displayData = ref('Loading...');
const candleChartContainer = ref(null);
const volumeChartContainer = ref(null);
const supertrendChartContainer = ref(null);
const wavetrendChartContainer = ref(null);
const rsiChartContainer = ref(null);
const macdChartContainer = ref(null);
const adxChartContainer = ref(null);
let candleChart, volumeChart, supertrendChart, wavetrendChart, rsiChart, macdChart, adxChart;
let candlestickSeries, volumeSeries, rsiSeries, macdLineSeries, signalLineSeries, histogramSeries, adxSeries, supertrendSeries, wt1Series, wt2Series;

// 폴더 내 파일 목록 가져오기
const dataFiles = import.meta.glob('/src/data/*.csv');
const dataFileNames = Object.keys(dataFiles).map((filePath) => filePath.replace('/src/data/', '').replace('.csv', ''));
const selectedDataFile = ref(dataFileNames[0]); // 기본 선택 파일

const markerSeries = ref([]); // markerSeries를 ref로 선언

// CSV 파일에서 데이터 불러오기
const loadCSV = async () => {
    try {
        const filePath = `/src/data/${selectedDataFile.value}.csv`;
        const response = await fetch(filePath);
        const text = await response.text();
        const csvData = d3.csvParse(text);

        data.value = csvData.map((row) => ({
            time: new Date(row.time).getTime() / 1000, // 시간 데이터를 타임스탬프로 변환
            open: parseFloat(row.open) || 0,
            high: parseFloat(row.high) || 0,
            low: parseFloat(row.low) || 0,
            close: parseFloat(row.close) || 0,
            volume: parseFloat(row.volume) || 0,
            supertrend: parseFloat(row.supertrend) || NaN,
            wt1: parseFloat(row.wt1) || NaN,
            wt2: parseFloat(row.wt2) || NaN,
            upperBand: parseFloat(row.upperband) || NaN,
            lowerBand: parseFloat(row.lowerband) || NaN,
            rsi: parseFloat(row.rsi) || NaN,
            adx: parseFloat(row.adx) || NaN,
            pdi: parseFloat(row.pdi) || NaN,
            mdi: parseFloat(row.mdi) || NaN,
            macd: parseFloat(row.macd) || NaN,
            macdSignal: parseFloat(row.macd_signal) || NaN,
            macdHist: parseFloat(row.macd_hist) || NaN
        }));
        // console.log(data.value);
        // 볼륨 데이터 설정
        volumeData.value = data.value.map((item) => ({
            time: item.time,
            value: item.volume,
            color: item.close >= item.open ? '#4caf50' : '#f44336'
        }));

        // Supertrend 데이터 설정

        // Supertrend 데이터를 상한선과 하한선을 기준으로 색상을 분리하여 설정
        supertrendData.value = data.value.map((item) => ({
            time: item.time,
            value: item.upperBand || item.lowerBand, // upperBand가 존재하면 upperBand, 아니면 lowerBand
            color: item.upperBand ? '#f44336' : '#4caf50' // upperBand가 있으면 빨간색, 없으면 초록색
        }));

        // wavetrend 데이터 설정
        wavetrendData.value = data.value.map((item) => ({
            time: item.time,
            wt1: item.wt1,
            wt2: item.wt2
        }));

        // RSI 데이터 설정
        rsiData.value = data.value.map((item) => ({
            time: item.time,
            value: item.rsi
        }));

        // MACD 데이터 설정
        macdData.value = data.value.map((item) => ({
            time: item.time,
            macd: item.macd,
            signal: item.macdSignal,
            histogram: item.macdHist
        }));

        // ADX 데이터 설정
        adxData.value = data.value.map((item) => ({
            time: item.time,
            adx: item.adx,
            pdi: item.pdi,
            mdi: item.mdi
        }));

        initializeCharts();
    } catch (error) {
        console.error('Error loading CSV data:', error);
    }
};
//
const chartChoice = ref('');
// 차트 초기화 및 데이터 설정
const initializeCharts = () => {
    if (candleChart) candleChart.remove();
    if (supertrendChart) supertrendChart.remove();

    // 첫 번째 차트 (캔들 차트) 생성
    candleChart = createChart(candleChartContainer.value, {
        width: candleChartContainer.value.clientWidth,
        height: 350,
        layout: { backgroundColor: '#FFFFFF', textColor: '#000' },
        grid: { vertLines: { color: '#e1e3e6' }, horzLines: { color: '#e1e3e6' } },
        timeScale: { borderColor: '#d1d4dc', timeVisible: true },
        crosshair: { mode: 0 } // crosshair 모드 설정
    });
    candlestickSeries = candleChart.addCandlestickSeries({
        upColor: '#4caf50',
        downColor: '#f44336',
        borderUpColor: '#4caf50',
        borderDownColor: '#f44336'
    });
    candlestickSeries.setData(data.value);

    // 두 번째 차트 (볼륨 차트) 생성
    volumeChart = createChart(volumeChartContainer.value, {
        width: volumeChartContainer.value.clientWidth,
        height: 100,
        layout: { backgroundColor: '#FFFFFF', textColor: '#000' },
        grid: { vertLines: { color: '#e1e3e6' }, horzLines: { color: '#e1e3e6' } },
        timeScale: { borderColor: '#d1d4dc', timeVisible: true },
        crosshair: { mode: 0 } // crosshair 모드 설정
    });
    volumeSeries = volumeChart.addHistogramSeries({
        color: (bar) => bar.color,
        priceFormat: { type: 'volume' }
    });
    volumeSeries.setData(volumeData.value);

    // Supertrend 차트 생성, long과 short를 표시하기 위해 두 개의 라인 시리즈 생성
    supertrendChart = createChart(supertrendChartContainer.value, {
        width: supertrendChartContainer.value.clientWidth,
        height: 200,
        layout: { backgroundColor: '#FFFFFF', textColor: '#000' },
        grid: { vertLines: { color: '#e1e3e6' }, horzLines: { color: '#e1e3e6' } },
        timeScale: { borderColor: '#d1d4dc', timeVisible: true }
    });

    // 각 구간을 색상에 맞춰 표시
    const supertrendSeries = supertrendChart.addLineSeries({
        color: (line) => line.color,
        lineWidth: 2
    });
    supertrendSeries.setData(supertrendData.value);

    // wavetrend 차트 생성
    wavetrendChart = createChart(wavetrendChartContainer.value, {
        width: wavetrendChartContainer.value.clientWidth,
        height: 200,
        layout: { backgroundColor: '#FFFFFF', textColor: '#000' },
        grid: { vertLines: { color: '#e1e3e6' }, horzLines: { color: '#e1e3e6' } },
        timeScale: { borderColor: '#d1d4dc', timeVisible: true }
    });
    // 각 구간을 색상에 맞춰 표시
    wt1Series = wavetrendChart.addLineSeries({
        color: 'blue',
        lineWidth: 1
    });
    wt2Series = wavetrendChart.addLineSeries({
        color: 'red',
        lineWidth: 1
    });
    wt1Series.setData(wavetrendData.value.map((item) => ({ time: item.time, value: item.wt1 })));
    wt2Series.setData(wavetrendData.value.map((item) => ({ time: item.time, value: item.wt2 })));

    // RSI 차트 생성
    rsiChart = createChart(rsiChartContainer.value, {
        width: rsiChartContainer.value.clientWidth,
        height: 200,
        layout: { backgroundColor: '#FFFFFF', textColor: '#000' },
        grid: { vertLines: { color: '#e1e3e6' }, horzLines: { color: '#e1e3e6' } },
        timeScale: { borderColor: '#d1d4dc', timeVisible: true }
    });
    rsiSeries = rsiChart.addLineSeries({
        color: 'violet',
        lineWidth: 1
    });
    rsiSeries.setData(rsiData.value);

    // MACD 차트 생성
    macdChart = createChart(macdChartContainer.value, {
        width: macdChartContainer.value.clientWidth,
        height: 200,
        layout: { backgroundColor: '#FFFFFF', textColor: '#000' },
        grid: { vertLines: { color: '#e1e3e6' }, horzLines: { color: '#e1e3e6' } },
        timeScale: { borderColor: '#d1d4dc', timeVisible: true }
    });

    // MACD 라인, 신호 라인, 히스토그램 시리즈 생성
    macdLineSeries = macdChart.addLineSeries({
        color: 'blue',
        lineWidth: 1
    });
    signalLineSeries = macdChart.addLineSeries({
        color: 'red',
        lineWidth: 1
    });
    histogramSeries = macdChart.addHistogramSeries({
        color: 'green',
        priceFormat: { type: 'volume' },
        scaleMargins: { top: 0.8, bottom: 0 } // 히스토그램의 위치 조정
    });

    // MACD, 신호, 히스토그램 데이터 설정
    macdLineSeries.setData(macdData.value.map((item) => ({ time: item.time, value: item.macd })));
    signalLineSeries.setData(macdData.value.map((item) => ({ time: item.time, value: item.signal })));
    histogramSeries.setData(macdData.value.map((item) => ({ time: item.time, value: item.histogram })));

    // ADX 차트 생성
    adxChart = createChart(adxChartContainer.value, {
        width: adxChartContainer.value.clientWidth,
        height: 200,
        layout: { backgroundColor: '#FFFFFF', textColor: '#000' },
        grid: { vertLines: { color: '#e1e3e6' }, horzLines: { color: '#e1e3e6' } },
        timeScale: { borderColor: '#d1d4dc', timeVisible: true }
    });

    // ADX, +DI, -DI 시리즈 생성
    adxSeries = adxChart.addLineSeries({
        color: 'red', // 빨간색 ADX
        lineWidth: 1
    });
    const pdiSeries = adxChart.addLineSeries({
        color: 'blue', // 파란색 PDI
        lineWidth: 1
    });
    const mdiSeries = adxChart.addLineSeries({
        color: 'orange', // 오랜지색 MDI
        lineWidth: 1
    });

    // ADX, +DI, -DI 데이터 설정
    adxSeries.setData(adxData.value.map((item) => ({ time: item.time, value: item.adx })));
    pdiSeries.setData(adxData.value.map((item) => ({ time: item.time, value: item.pdi })));
    mdiSeries.setData(adxData.value.map((item) => ({ time: item.time, value: item.mdi })));

    onUnmounted(() => {
        resizeObserver.disconnect();
        candleChart.remove();
        supertrendChart.remove();
        wavetrendChart.remove();
        volumeChart.remove();
        rsiChart.remove();
        macdChart.remove();
        adxChart.remove();
    });

    candleChart.timeScale().subscribeVisibleLogicalRangeChange((timeRange) => {
        volumeChart.timeScale().setVisibleLogicalRange(timeRange);
        supertrendChart.timeScale().setVisibleLogicalRange(timeRange);
        wavetrendChart.timeScale().setVisibleLogicalRange(timeRange);
        rsiChart.timeScale().setVisibleLogicalRange(timeRange);
        macdChart.timeScale().setVisibleLogicalRange(timeRange);
        adxChart.timeScale().setVisibleLogicalRange(timeRange);
    });

    volumeChart.timeScale().subscribeVisibleLogicalRangeChange((timeRange) => {
        candleChart.timeScale().setVisibleLogicalRange(timeRange);
        supertrendChart.timeScale().setVisibleLogicalRange(timeRange);
        wavetrendChart.timeScale().setVisibleLogicalRange(timeRange);
        rsiChart.timeScale().setVisibleLogicalRange(timeRange);
        macdChart.timeScale().setVisibleLogicalRange(timeRange);
        adxChart.timeScale().setVisibleLogicalRange(timeRange);
    });

    supertrendChart.timeScale().subscribeVisibleLogicalRangeChange((timeRange) => {
        candleChart.timeScale().setVisibleLogicalRange(timeRange);
        volumeChart.timeScale().setVisibleLogicalRange(timeRange);
        rsiChart.timeScale().setVisibleLogicalRange(timeRange);
        macdChart.timeScale().setVisibleLogicalRange(timeRange);
        adxChart.timeScale().setVisibleLogicalRange(timeRange);
        wavetrendChart.timeScale().setVisibleLogicalRange(timeRange);
    });

    wavetrendChart.timeScale().subscribeVisibleLogicalRangeChange((timeRange) => {
        candleChart.timeScale().setVisibleLogicalRange(timeRange);
        volumeChart.timeScale().setVisibleLogicalRange(timeRange);
        supertrendChart.timeScale().setVisibleLogicalRange(timeRange);
        rsiChart.timeScale().setVisibleLogicalRange(timeRange);
        macdChart.timeScale().setVisibleLogicalRange(timeRange);
        adxChart.timeScale().setVisibleLogicalRange(timeRange);
    });

    rsiChart.timeScale().subscribeVisibleLogicalRangeChange((timeRange) => {
        candleChart.timeScale().setVisibleLogicalRange(timeRange);
        volumeChart.timeScale().setVisibleLogicalRange(timeRange);
        supertrendChart.timeScale().setVisibleLogicalRange(timeRange);
        wavetrendChart.timeScale().setVisibleLogicalRange(timeRange);
        macdChart.timeScale().setVisibleLogicalRange(timeRange);
        adxChart.timeScale().setVisibleLogicalRange(timeRange);
    });

    macdChart.timeScale().subscribeVisibleLogicalRangeChange((timeRange) => {
        candleChart.timeScale().setVisibleLogicalRange(timeRange);
        volumeChart.timeScale().setVisibleLogicalRange(timeRange);
        supertrendChart.timeScale().setVisibleLogicalRange(timeRange);
        wavetrendChart.timeScale().setVisibleLogicalRange(timeRange);
        rsiChart.timeScale().setVisibleLogicalRange(timeRange);
        adxChart.timeScale().setVisibleLogicalRange(timeRange);
    });

    adxChart.timeScale().subscribeVisibleLogicalRangeChange((timeRange) => {
        candleChart.timeScale().setVisibleLogicalRange(timeRange);
        volumeChart.timeScale().setVisibleLogicalRange(timeRange);
        supertrendChart.timeScale().setVisibleLogicalRange(timeRange);
        wavetrendChart.timeScale().setVisibleLogicalRange(timeRange);
        rsiChart.timeScale().setVisibleLogicalRange(timeRange);
        macdChart.timeScale().setVisibleLogicalRange(timeRange);
    });

    // // 차트 간의 crosshair 동기화
    candleChart.subscribeCrosshairMove((param) => {
        chartChoice.value = 'candle';
        const dataPoint = getCrosshairDataPoint(candlestickSeries, param);
        syncCrosshair(volumeChart, volumeSeries, dataPoint);
        syncCrosshair(supertrendChart, supertrendSeries, dataPoint);
        syncCrosshair(wavetrendChart, wt1Series, dataPoint);
        syncCrosshair(rsiChart, rsiSeries, dataPoint);
        syncCrosshair(macdChart, macdLineSeries, dataPoint);
        syncCrosshair(adxChart, adxSeries, dataPoint);
    });

    volumeChart.subscribeCrosshairMove((param) => {
        chartChoice.value = 'volume';
        const dataPoint = getCrosshairDataPoint(volumeSeries, param);
        syncCrosshair(candleChart, candlestickSeries, dataPoint);
        syncCrosshair(supertrendChart, supertrendSeries, dataPoint);
        syncCrosshair(wavetrendChart, wt1Series, dataPoint);
        syncCrosshair(rsiChart, rsiSeries, dataPoint);
        syncCrosshair(macdChart, macdLineSeries, dataPoint);
        syncCrosshair(adxChart, adxSeries, dataPoint);
    });
    supertrendChart.subscribeCrosshairMove((param) => {
        chartChoice.value = 'supertrend';
        const dataPoint = getCrosshairDataPoint(supertrendSeries, param);
        syncCrosshair(candleChart, candlestickSeries, dataPoint);
        syncCrosshair(volumeChart, volumeSeries, dataPoint);
        syncCrosshair(wavetrendChart, wt1Series, dataPoint);
        syncCrosshair(rsiChart, rsiSeries, dataPoint);
        syncCrosshair(macdChart, macdLineSeries, dataPoint);
        syncCrosshair(adxChart, adxSeries, dataPoint);
    });
    wavetrendChart.subscribeCrosshairMove((param) => {
        chartChoice.value = 'wavetrend';
        const dataPoint = getCrosshairDataPoint(wt1Series, param);
        syncCrosshair(candleChart, candlestickSeries, dataPoint);
        syncCrosshair(volumeChart, volumeSeries, dataPoint);
        syncCrosshair(supertrendChart, supertrendSeries, dataPoint);
        syncCrosshair(rsiChart, rsiSeries, dataPoint);
        syncCrosshair(macdChart, macdLineSeries, dataPoint);
        syncCrosshair(adxChart, adxSeries, dataPoint);
    });
    rsiChart.subscribeCrosshairMove((param) => {
        chartChoice.value = 'rsi';
        const dataPoint = getCrosshairDataPoint(rsiSeries, param);
        syncCrosshair(candleChart, candlestickSeries, dataPoint);
        syncCrosshair(volumeChart, volumeSeries, dataPoint);
        syncCrosshair(supertrendChart, supertrendSeries, dataPoint);
        syncCrosshair(wavetrendChart, wt1Series, dataPoint);
        syncCrosshair(macdChart, macdLineSeries, dataPoint);
        syncCrosshair(adxChart, adxSeries, dataPoint);
    });
    macdChart.subscribeCrosshairMove((param) => {
        chartChoice.value = 'macd';
        const dataPoint = getCrosshairDataPoint(macdLineSeries, param);
        syncCrosshair(candleChart, candlestickSeries, dataPoint);
        syncCrosshair(volumeChart, volumeSeries, dataPoint);
        syncCrosshair(supertrendChart, supertrendSeries, dataPoint);
        syncCrosshair(wavetrendChart, wt1Series, dataPoint);
        syncCrosshair(rsiChart, rsiSeries, dataPoint);
        syncCrosshair(adxChart, adxSeries, dataPoint);
    });
    adxChart.subscribeCrosshairMove((param) => {
        chartChoice.value = 'adx';
        const dataPoint = getCrosshairDataPoint(adxSeries, param);
        syncCrosshair(candleChart, candlestickSeries, dataPoint);
        syncCrosshair(volumeChart, volumeSeries, dataPoint);
        syncCrosshair(supertrendChart, supertrendSeries, dataPoint);
        syncCrosshair(wavetrendChart, wt1Series, dataPoint);
        syncCrosshair(rsiChart, rsiSeries, dataPoint);
        syncCrosshair(macdChart, macdLineSeries, dataPoint);
    });
};
function getCrosshairDataPoint(series, param) {
    if (!param.time) {
        return null;
    }
    const dataPoint = param.seriesData.get(series);
    handleCrosshairMove(param.seriesData.get(series).time); // 필요한 데이터 포맷에 맞춰 사용

    return dataPoint || null;
}

function syncCrosshair(chart, series, dataPoint) {
    if (dataPoint) {
        chart.setCrosshairPosition(dataPoint.value, dataPoint.time, series);
        return;
    }
    chart.clearCrosshairPosition();
}
// crosshairMove 이벤트 핸들러
function handleCrosshairMove(param) {
    const data2 = data.value.find((item) => item.time === param);
    if (!data2) return;

    // console.log('Hovered Data:', data2);
    // console.log('Current chartChoice:', chartChoice.value);

    const isHighlighted = (key) => chartChoice.value === key;

    displayData.value = `
        <span class="${isHighlighted('candle') ? 'chartChoice' : ''}">Open: ${data2.open}</span> | 
        <span class="${isHighlighted('candle') ? 'chartChoice' : ''}">High: ${data2.high}</span> | 
        <span class="${isHighlighted('candle') ? 'chartChoice' : ''}">Low: ${data2.low}</span> | 
        <span class="${isHighlighted('candle') ? 'chartChoice' : ''}">Close: ${data2.close}</span> | 
        <span class="${isHighlighted('volume') ? 'chartChoice' : ''}">Volume: ${data2.volume}</span> | 
        <span class="${isHighlighted('supertrend') ? 'chartChoice' : ''}">Supertrend: ${data2.supertrend}</span><br>
        <span class="${isHighlighted('wavetrend') ? 'chartChoice' : ''}">WT1: ${data2.wt1}</span> | 
        <span class="${isHighlighted('wavetrend') ? 'chartChoice' : ''}">WT2: ${data2.wt2}</span> | 
        <span class="${isHighlighted('rsi') ? 'chartChoice' : ''}">RSI: ${data2.rsi}</span> | 
        <span class="${isHighlighted('macd') ? 'chartChoice' : ''}">MACD: ${data2.macd}</span> | 
        <span class="${isHighlighted('macd') ? 'chartChoice' : ''}">Signal: ${data2.macdSignal}</span> | 
        <span class="${isHighlighted('macd') ? 'chartChoice' : ''}">Histogram: ${data2.macdHist}</span> | 
        <span class="${isHighlighted('adx') ? 'chartChoice' : ''}">ADX: ${data2.adx}</span> | 
        <span class="${isHighlighted('adx') ? 'chartChoice' : ''}">PDI: ${data2.pdi}</span> | 
        <span class="${isHighlighted('adx') ? 'chartChoice' : ''}">MDI: ${data2.mdi}</span>
    `;
}

// 초기 마운트 시 CSV 파일 로드
onMounted(() => {
    loadCSV();
});
</script>

<style>
.p-inputnumber input {
    width: 70px;
}
.chart-container {
    display: flex;
    flex-direction: column;
    gap: 20px;
}
.top-chart {
    order: -1; /* 가장 위로 이동 */
}
.firstchart {
    order: -1; /* 항상 첫번째 */
}
.chartChoice {
    color: red;
}
.p-scrollpanel-content {
    padding: 0 18px 18px 18px;
}
</style>

