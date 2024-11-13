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

        <div>{{ displayData }}</div>
        <!-- 항상 표시될 데이터 위치 -->

        <div class="flex">
            <div style="width: 80%">
                <!-- 차트 컨테이너 -->
                <div ref="candleChartContainer" style="width: 100%; height: 400px; margin-bottom: 20px"></div>
                <div ref="volumeChartContainer" style="width: 100%; height: 100px; margin-bottom: 20px"></div>
                <div ref="supertrendChartContainer" style="width: 100%; height: 100px; margin-bottom: 20px"></div>
                <div ref="rsiChartContainer" style="width: 100%; height: 100px; margin-bottom: 20px"></div>
                <div ref="macdChartContainer" style="width: 100%; height: 150px; margin-bottom: 20px"></div>
                <div ref="adxChartContainer" style="width: 100%; height: 200px"></div>
            </div>
            <div style="width: 20%">
                <InputNumber v-model="inputValue" @update:modelValue="findData" placeholder="ADX 값 입력" />
                {{ inputValue }}
                <div>조건</div>
                <div class="card flex-row">
                    <div class="flex align-content-center mb-2">
                        <div class="mr-2">SuperTrend Long</div>
                        <Checkbox v-model="supertrendLong" :binary="true"></Checkbox>
                    </div>

                    <div class="flex align-content-center">
                        <div class="mr-2">SuperTrend Short</div>
                        <Checkbox v-model="supertrendShort" :binary="true"></Checkbox>
                    </div>
                </div>
                <div class="card flex-row">
                    <div>RSI</div>
                    <div><InputNumber v-model="rsiOver" /> 이상일때</div>
                    <div><InputNumber v-model="rsiUnder" /> 이하일때</div>
                </div>
                <div class="card flex-row">
                    <div>MACD</div>
                    <div style="color: blue">MACD<InputNumber v-model="macdOver"></InputNumber>이상일때</div>
                    <div style="color: blue">MACD<InputNumber v-model="macdUnder"></InputNumber>이하일때</div>

                    <div style="color: red">Signal<InputNumber v-model="signalUnder"></InputNumber>이하일때</div>
                    <div style="color: red">Signal<InputNumber v-model="signalOver"></InputNumber>이상일때</div>
                    <div class="mt-2">
                        <div class="flex align-content-center mb-2">
                            <div style="color: blue" class="mr-2">MACD가Signal보다 클때</div>
                            <div><Checkbox v-model="macdMore" :binary="true"></Checkbox></div>
                        </div>
                        <div class="flex align-content-center mb-2">
                            <div style="color: red" class="mr-2">Signal가MACD보다 클때</div>
                            <div><Checkbox v-model="signalMore" :binary="true"></Checkbox></div>
                        </div>
                    </div>
                </div>
                <div class="card flex">
                    <OrderList v-model="adxindicatorsbefore" :dragdrop="true" header="Indicator Order">
                        <template #header>직전자료</template>
                        <template #item="slotProps">
                            <div class="p-d-flex p-ai-center p-jc-between" style="width: 100%">
                                <div>{{ slotProps.index + 1 }}. {{ slotProps.item.name }}</div>
                                <!-- 순번을 표시 -->
                            </div>
                        </template>
                    </OrderList>

                    <OrderList v-model="adxindicatorsafter" :dragdrop="true" header="Indicator Order">
                        <template #header>바뀐자료</template>
                        <template #item="slotProps">
                            <div class="p-d-flex p-ai-center p-jc-between" style="width: 100%">
                                <div>{{ slotProps.index + 1 }}. {{ slotProps.item.name }}</div>
                            </div>
                        </template>
                    </OrderList>
                </div>

                <div class="card flex-row">
                    <div>ADX</div>
                    <div style="color: red">ADX <InputNumber /></div>
                    <div style="color: blue">+DI<InputNumber></InputNumber>이상일때</div>
                    <div style="color: orange">-DI<InputNumber></InputNumber>이상일때</div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue';
import { createChart } from 'lightweight-charts';
import * as d3 from 'd3';

// 조건관련 변수
const supertrendLong = ref(false);
const supertrendShort = ref(false);
const rsiOver = ref(0);
const rsiUnder = ref(0);
const macdOver = ref(0);
const macdUnder = ref(0);
const signalUnder = ref(0);
const signalOver = ref(0);
const macdMore = ref(false);
const signalMore = ref(false);
const adxvalue = ref(0);
// Indicator 리스트
const adxindicatorsbefore = ref([{ name: 'ADX' }, { name: '+DI' }, { name: '-DI' }]);
const adxindicatorsafter = ref([{ name: 'ADX' }, { name: '+DI' }, { name: '-DI' }]);

// 데이터 관련 변수
const inputValue = ref('');
const data = ref([]);
const volumeData = ref([]);
const supertrendData = ref([]);
const supertrendDataFormatted = ref([]);
const rsiData = ref([]);
const macdData = ref([]);
const adxData = ref([]);
const displayData = ref('Loading...');
const candleChartContainer = ref(null);
const volumeChartContainer = ref(null);
const supertrendChartContainer = ref(null);
const rsiChartContainer = ref(null);
const macdChartContainer = ref(null);
const adxChartContainer = ref(null);
let candleChart, volumeChart, supertrendChart, rsiChart, macdChart, adxChart;
let candlestickSeries, volumeSeries, supertrendSeries, rsiSeries, macdLineSeries, signalLineSeries, histogramSeries, adxSeries;

// 폴더 내 파일 목록 가져오기
const dataFiles = import.meta.glob('/src/data/*.csv');
const dataFileNames = Object.keys(dataFiles).map((filePath) => filePath.replace('/src/data/', '').replace('.csv', ''));
const selectedDataFile = ref(dataFileNames[0]); // 기본 선택 파일

const markerSeries = ref([]); // markerSeries를 ref로 선언

// Open 값이 입력된 값 이상인 데이터 검색 및 마커 추가
const findData = () => {
    const targetOpenValue = parseFloat(inputValue.value);
    if (isNaN(targetOpenValue)) return;

    // 조건에 맞는 모든 데이터에 마커 추가
    markerSeries.value = data.value
        .filter((item) => item.open >= targetOpenValue)
        .map((item) => ({
            time: item.time,
            position: 'aboveBar',
            color: 'blue',
            shape: 'arrowDown'
        }));

    // 차트에 마커 추가
    candlestickSeries.setMarkers(markerSeries.value);
};
// MACD 골든 크로스 감지 및 마커 추가
const findGoldenCross = () => {
    // 골든 크로스 감지 조건에 맞는 데이터 필터링
    markerSeries.value = data.value
        .filter((item, index, array) => {
            if (index === 0) return false; // 첫 번째 항목은 이전 데이터가 없으므로 패스

            const prevItem = array[index - 1];
            return prevItem.macd <= prevItem.macdSignal && item.macd > item.macdSignal;
        })
        .map((item) => ({
            time: item.time,
            position: 'aboveBar',
            color: 'blue',
            shape: 'arrowUp',
            text: `Golden Cross at ${item.time}`
        }));

    // 차트에 마커 추가
    candlestickSeries.setMarkers(markerSeries.value);
};
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

        // console.log(supertrendData.value);

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

// 차트 초기화 및 데이터 설정
const initializeCharts = () => {
    if (candleChart) candleChart.remove();
    if (supertrendChart) supertrendChart.remove();

    // 첫 번째 차트 (캔들 차트) 생성
    candleChart = createChart(candleChartContainer.value, {
        width: candleChartContainer.value.clientWidth,
        height: 400,
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
        height: 100,
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

    // RSI 차트 생성
    rsiChart = createChart(rsiChartContainer.value, {
        width: rsiChartContainer.value.clientWidth,
        height: 100,
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
        height: 150,
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
        color: 'blue', // 파란색 +DI
        lineWidth: 1
    });
    const mdiSeries = adxChart.addLineSeries({
        color: 'orange', // 오랜지색 -DI
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
        volumeChart.remove();
        rsiChart.remove();
        macdChart.remove();
        adxChart.remove();
    });

    candleChart.timeScale().subscribeVisibleLogicalRangeChange((timeRange) => {
        volumeChart.timeScale().setVisibleLogicalRange(timeRange);
        supertrendChart.timeScale().setVisibleLogicalRange(timeRange);
        rsiChart.timeScale().setVisibleLogicalRange(timeRange);
        macdChart.timeScale().setVisibleLogicalRange(timeRange);
        adxChart.timeScale().setVisibleLogicalRange(timeRange);
    });

    volumeChart.timeScale().subscribeVisibleLogicalRangeChange((timeRange) => {
        candleChart.timeScale().setVisibleLogicalRange(timeRange);
        supertrendChart.timeScale().setVisibleLogicalRange(timeRange);
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
    });
    rsiChart.timeScale().subscribeVisibleLogicalRangeChange((timeRange) => {
        candleChart.timeScale().setVisibleLogicalRange(timeRange);
        volumeChart.timeScale().setVisibleLogicalRange(timeRange);
        supertrendChart.timeScale().setVisibleLogicalRange(timeRange);
        macdChart.timeScale().setVisibleLogicalRange(timeRange);
        adxChart.timeScale().setVisibleLogicalRange(timeRange);
    });
    macdChart.timeScale().subscribeVisibleLogicalRangeChange((timeRange) => {
        candleChart.timeScale().setVisibleLogicalRange(timeRange);
        volumeChart.timeScale().setVisibleLogicalRange(timeRange);
        supertrendChart.timeScale().setVisibleLogicalRange(timeRange);
        rsiChart.timeScale().setVisibleLogicalRange(timeRange);
        adxChart.timeScale().setVisibleLogicalRange(timeRange);
    });
    adxChart.timeScale().subscribeVisibleLogicalRangeChange((timeRange) => {
        candleChart.timeScale().setVisibleLogicalRange(timeRange);
        volumeChart.timeScale().setVisibleLogicalRange(timeRange);
        supertrendChart.timeScale().setVisibleLogicalRange(timeRange);
        rsiChart.timeScale().setVisibleLogicalRange(timeRange);
        macdChart.timeScale().setVisibleLogicalRange(timeRange);
    });
    candleChart.subscribeCrosshairMove((param) => {
        const dataPoint = getCrosshairDataPoint(candlestickSeries, param);
        syncCrosshair(volumeChart, volumeSeries, dataPoint);
        syncCrosshair(supertrendChart, supertrendSeries, dataPoint);
        syncCrosshair(rsiChart, rsiSeries, dataPoint);
        syncCrosshair(macdChart, macdLineSeries, dataPoint);
        syncCrosshair(adxChart, adxSeries, dataPoint);
    });

    volumeChart.subscribeCrosshairMove((param) => {
        const dataPoint = getCrosshairDataPoint(volumeSeries, param);
        syncCrosshair(candleChart, candlestickSeries, dataPoint);
        syncCrosshair(supertrendChart, supertrendSeries, dataPoint);
        syncCrosshair(rsiChart, rsiSeries, dataPoint);
        syncCrosshair(macdChart, macdLineSeries, dataPoint);
        syncCrosshair(adxChart, adxSeries, dataPoint);
    });
    supertrendChart.subscribeCrosshairMove((param) => {
        const dataPoint = getCrosshairDataPoint(supertrendSeries, param);
        syncCrosshair(candleChart, candlestickSeries, dataPoint);
        syncCrosshair(volumeChart, volumeSeries, dataPoint);
        syncCrosshair(rsiChart, rsiSeries, dataPoint);
        syncCrosshair(macdChart, macdLineSeries, dataPoint);
        syncCrosshair(adxChart, adxSeries, dataPoint);
    });
    rsiChart.subscribeCrosshairMove((param) => {
        const dataPoint = getCrosshairDataPoint(rsiSeries, param);
        syncCrosshair(candleChart, candlestickSeries, dataPoint);
        syncCrosshair(volumeChart, volumeSeries, dataPoint);
        syncCrosshair(supertrendChart, supertrendSeries, dataPoint);
        syncCrosshair(macdChart, macdLineSeries, dataPoint);
        syncCrosshair(adxChart, adxSeries, dataPoint);
    });
    macdChart.subscribeCrosshairMove((param) => {
        const dataPoint = getCrosshairDataPoint(macdLineSeries, param);
        syncCrosshair(candleChart, candlestickSeries, dataPoint);
        syncCrosshair(volumeChart, volumeSeries, dataPoint);
        syncCrosshair(supertrendChart, supertrendSeries, dataPoint);
        syncCrosshair(rsiChart, rsiSeries, dataPoint);
        syncCrosshair(adxChart, adxSeries, dataPoint);
    });
    adxChart.subscribeCrosshairMove((param) => {
        const dataPoint = getCrosshairDataPoint(adxSeries, param);
        syncCrosshair(candleChart, candlestickSeries, dataPoint);
        syncCrosshair(volumeChart, volumeSeries, dataPoint);
        syncCrosshair(supertrendChart, supertrendSeries, dataPoint);
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
    displayData.value = `Open: ${data2.open} | High: ${data2.high} | Low: ${data2.low} | Close: ${data2.close} | Volume: ${data2.volume} | Supertrend: ${data2.supertrend} | RSI: ${data2.rsi} | MACD: ${data2.macd} | Signal: ${data2.macdSignal} | Histogram: ${data2.macdHist} | ADX: ${data2.adx} | PDI: ${data2.pdi} | MDI: ${data2.mdi}`;
    return;
}

// 초기 마운트 시 CSV 파일 로드
onMounted(() => {
    loadCSV();
});
</script>

<style>
/* 필요한 스타일 추가 */
</style>

