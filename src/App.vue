<template>
    <div>
        <h1>Bybit Kline Data with Data Tooltip</h1>
        <div>{{ displayData }}</div>
        <!-- 항상 표시될 데이터 위치 -->
        <!-- 파일 선택 드롭다운 -->
        <label for="file-select">Select Data File: </label>
        <select id="file-select" v-model="selectedDataFile" @change="loadCSV">
            <option v-for="fileName in dataFileNames" :key="fileName" :value="fileName">
                {{ fileName }}
            </option>
        </select>

        <!-- 지표 체크박스 -->
        <label> <input type="checkbox" v-model="showVolumeChart" /> Show Volume Chart </label>
        <label> <input type="checkbox" v-model="showMACDChart" /> Show MACD Chart </label>

        <!-- 차트 컨테이너 -->
        <div ref="candleChartContainer" style="width: 100%; height: 300px; margin-bottom: 20px"></div>
        <div v-if="showVolumeChart" ref="volumeChartContainer" style="width: 100%; height: 100px; margin-bottom: 20px"></div>
        <div v-if="showMACDChart" ref="macdChartContainer" style="width: 100%; height: 200px"></div>
    </div>
</template>
  
  <script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import { createChart } from 'lightweight-charts';
import * as d3 from 'd3';
// Import MACD from technicalindicators
import { MACD } from 'technicalindicators';

// 데이터 관련 변수
const data = ref([]);
const volumeData = ref([]);
const macdData = ref([]);
const displayData = ref('Loading...'); // 항상 표시될 데이터
const candleChartContainer = ref(null);
const volumeChartContainer = ref(null);
const macdChartContainer = ref(null);
let candleChart;
let volumeChart;
let macdChart;
let candlestickSeries;
let volumeSeries;
let macdLineSeries;
let signalLineSeries;
let histogramSeries;

// 지표 체크박스 상태
const showVolumeChart = ref(true);
const showMACDChart = ref(true);

// MACD 계산 함수
const calculateMACD = (closePrices) => {
    const macdInput = {
        values: closePrices,
        fastPeriod: 12,
        slowPeriod: 26,
        signalPeriod: 9,
        SimpleMAOscillator: false,
        SimpleMASignal: false
    };
    const macd = MACD.calculate(macdInput);

    if (macd.length > 0) {
        macdData.value = macd
            .map((point, index) => ({
                time: data.value[index + 26]?.time, // slowPeriod 이후부터 시작
                macd: point.MACD,
                signal: point.signal,
                histogram: point.histogram
            }))
            .filter((item) => item.time !== undefined); // undefined 제거
    }
};

// 폴더 내 파일 목록 가져오기
const dataFiles = import.meta.glob('/src/data/*.csv');
const dataFileNames = Object.keys(dataFiles).map((filePath) => filePath.replace('/src/data/', '').replace('.csv', ''));
const selectedDataFile = ref(dataFileNames[0]); // 기본 선택 파일

// CSV 파일에서 데이터 불러오기
const loadCSV = async () => {
    try {
        const filePath = `/src/data/${selectedDataFile.value}.csv`;
        const response = await fetch(filePath);
        const text = await response.text();
        const csvData = d3.csvParse(text);

        data.value = csvData.map((row) => ({
            time: +row.time / 1000,
            open: +row.open,
            high: +row.high,
            low: +row.low,
            close: +row.close,
            volume: +row.volume
        }));

        // 볼륨 데이터 설정
        volumeData.value = data.value.map((item) => ({
            time: item.time,
            value: item.volume,
            color: item.close >= item.open ? '#4caf50' : '#f44336'
        }));

        // MACD 계산
        calculateMACD(data.value.map((item) => item.close));
        initializeCharts();
    } catch (error) {
        console.error('Error loading CSV data:', error);
    }
};

// 차트 초기화 및 데이터 설정
const initializeCharts = () => {
    if (candleChart) candleChart.remove();
    if (volumeChart) volumeChart.remove();
    if (macdChart) macdChart.remove();

    // 첫 번째 차트 (캔들 차트) 생성
    candleChart = createChart(candleChartContainer.value, {
        width: candleChartContainer.value.clientWidth,
        height: 300,
        layout: { backgroundColor: '#FFFFFF', textColor: '#000' },
        grid: { vertLines: { color: '#e1e3e6' }, horzLines: { color: '#e1e3e6' } },
        timeScale: { borderColor: '#d1d4dc', timeVisible: true }
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
        timeScale: { visible: false }
    });
    volumeSeries = volumeChart.addHistogramSeries({ priceFormat: { type: 'volume' }, scaleMargins: { top: 0.8, bottom: 0 } });
    volumeSeries.setData(volumeData.value);

    // 세 번째 차트 (MACD 차트) 생성
    macdChart = createChart(macdChartContainer.value, {
        width: macdChartContainer.value.clientWidth,
        height: 200,
        layout: { backgroundColor: '#FFFFFF', textColor: '#000' },
        grid: { vertLines: { color: '#e1e3e6' }, horzLines: { color: '#e1e3e6' } },
        timeScale: { borderColor: '#d1d4dc', timeVisible: true }
    });
    macdLineSeries = macdChart.addLineSeries({ color: 'blue', lineWidth: 1 });
    signalLineSeries = macdChart.addLineSeries({ color: 'red', lineWidth: 1 });
    histogramSeries = macdChart.addHistogramSeries({ color: 'green' });

    macdLineSeries.setData(macdData.value.map((item) => ({ time: item.time, value: item.macd })));
    signalLineSeries.setData(macdData.value.map((item) => ({ time: item.time, value: item.signal })));
    histogramSeries.setData(macdData.value.map((item) => ({ time: item.time, value: item.histogram })));

    synchronizeCrosshair();
    synchronizeTimeScales();

    const resizeObserver = new ResizeObserver(() => {
        candleChart.applyOptions({ width: candleChartContainer.value.clientWidth });
        volumeChart.applyOptions({ width: volumeChartContainer.value.clientWidth });
        macdChart.applyOptions({ width: macdChartContainer.value.clientWidth });
    });
    resizeObserver.observe(candleChartContainer.value);
    resizeObserver.observe(volumeChartContainer.value);
    resizeObserver.observe(macdChartContainer.value);

    onUnmounted(() => {
        resizeObserver.disconnect();
        candleChart.remove();
        volumeChart.remove();
        macdChart.remove();
    });
};

// 시간 동기화
const synchronizeTimeScales = () => {
    const mainTimeScale = candleChart.timeScale();
    const volumeTimeScale = volumeChart.timeScale();
    const macdTimeScale = macdChart.timeScale();

    let isUpdating = false;

    const syncTimeScale = (source, target) => {
        source.subscribeVisibleTimeRangeChange((visibleRange) => {
            if (isUpdating) return;
            isUpdating = true;
            target.setVisibleRange(visibleRange);
            isUpdating = false;
        });
    };

    syncTimeScale(mainTimeScale, volumeTimeScale);
    syncTimeScale(mainTimeScale, macdTimeScale);
    syncTimeScale(volumeTimeScale, mainTimeScale);
    syncTimeScale(macdTimeScale, mainTimeScale);
};

// 수직 지시선 동기화

const synchronizeCrosshair = () => {
    const handleCrosshairMove = (param) => {
        if (param && param.time) {
            // 현재 시각에 맞는 데이터를 찾기
            const currentTime = param.time;
            const candlestick = data.value.find((item) => item.time === currentTime) || {};
            const volume = volumeData.value.find((item) => item.time === currentTime) || {};
            const macd = macdData.value.find((item) => item.time === currentTime) || {};

            // 모든 차트에서 동일하게 표시할 데이터 설정
            displayData.value = `${selectedDataFile.value} | Open: ${candlestick.open || 0} | High: ${candlestick.high || 0} | Low: ${candlestick.low || 0} | Close: ${candlestick.close || 0} | Volume: ${volume.value || 0}`;
            displayData.value += ` | MACD: ${macd.macd?.toFixed(2) || 0} | Signal: ${macd.signal?.toFixed(2) || 0} | Histogram: ${macd.histogram?.toFixed(2) || 0}`;
        }
    };

    // 각 차트에서 동일한 crosshair 이벤트 핸들러 적용
    candleChart.subscribeCrosshairMove(handleCrosshairMove);
    volumeChart.subscribeCrosshairMove(handleCrosshairMove);
    macdChart.subscribeCrosshairMove(handleCrosshairMove);
};

// 초기 마운트 시 CSV 파일 로드
onMounted(() => {
    loadCSV();
});
</script>
  
  <style>
/* 필요한 스타일 추가 */
</style>
  