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

        <!-- 차트 컨테이너 -->
        <div ref="candleChartContainer" style="width: 100%; height: 300px; margin-bottom: 20px"></div>
        <div ref="volumeChartContainer" style="width: 100%; height: 100px; margin-bottom: 20px"></div>
        <div ref="supertrendChartContainer" style="width: 100%; height: 100px; margin-bottom: 20px"></div>
        <div ref="rsiChartContainer" style="width: 100%; height: 100px; margin-bottom: 20px"></div>
        <div ref="macdChartContainer" style="width: 100%; height: 150px; margin-bottom: 20px"></div>
        <div ref="adxChartContainer" style="width: 100%; height: 100px"></div>
    </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import { createChart } from 'lightweight-charts';
import * as d3 from 'd3';

// 데이터 관련 변수
const data = ref([]);
const volumeData = ref([]);
const supertrendData = ref([]);
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
            volume: +row.volume,
            supertrend: +row.supertrend,
            upperBand: +row.upper_band,
            lowerBand: +row.lower_band,
            rsi: +row.rsi,
            macd: +row['blue(macd)'],
            macdSignal: +row['red(macd_signal)'],
            macdHist: +row['green(macd_hist)'],
            adx: +row['blue(adx)'],
            pdi: +row['green(pdi)'],
            mdi: +row['red(mdi)']
        }));

        // 볼륨 데이터 설정
        volumeData.value = data.value.map((item) => ({
            time: item.time,
            value: item.volume,
            color: item.close >= item.open ? '#4caf50' : '#f44336'
        }));

        // Supertrend 데이터 설정
        supertrendData.value = data.value.map((item) => ({
            time: item.time,
            value: item.supertrend,
            upperBand: item.upperBand,
            lowerBand: item.lowerBand
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

// 차트 초기화 및 데이터 설정
const initializeCharts = () => {
    if (candleChart) candleChart.remove();
    if (volumeChart) volumeChart.remove();
    if (supertrendChart) supertrendChart.remove();
    if (rsiChart) rsiChart.remove();
    if (macdChart) macdChart.remove();
    if (adxChart) adxChart.remove();

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

    // Supertrend 차트 생성
    supertrendChart = createChart(supertrendChartContainer.value, {
        width: supertrendChartContainer.value.clientWidth,
        height: 100,
        layout: { backgroundColor: '#FFFFFF', textColor: '#000' },
        grid: { vertLines: { color: '#e1e3e6' }, horzLines: { color: '#e1e3e6' } }
    });
    supertrendSeries = supertrendChart.addLineSeries({ color: 'green', lineWidth: 2 });
    supertrendSeries.setData(supertrendData.value.map((item) => ({ time: item.time, value: item.value })));

    // RSI 차트 생성
    rsiChart = createChart(rsiChartContainer.value, {
        width: rsiChartContainer.value.clientWidth,
        height: 100,
        layout: { backgroundColor: '#FFFFFF', textColor: '#000' },
        grid: { vertLines: { color: '#e1e3e6' }, horzLines: { color: '#e1e3e6' } }
    });
    rsiSeries = rsiChart.addLineSeries({ color: 'purple', lineWidth: 2 });
    rsiSeries.setData(rsiData.value);

    // MACD 차트 생성
    macdChart = createChart(macdChartContainer.value, {
        width: macdChartContainer.value.clientWidth,
        height: 150,
        layout: { backgroundColor: '#FFFFFF', textColor: '#000' },
        grid: { vertLines: { color: '#e1e3e6' }, horzLines: { color: '#e1e3e6' } }
    });
    macdLineSeries = macdChart.addLineSeries({ color: 'blue', lineWidth: 1 });
    signalLineSeries = macdChart.addLineSeries({ color: 'red', lineWidth: 1 });
    histogramSeries = macdChart.addHistogramSeries({ color: 'green' });

    // MACD 데이터 설정
    macdLineSeries.setData(macdData.value.map((item) => ({ time: item.time, value: item.macd })));
    signalLineSeries.setData(macdData.value.map((item) => ({ time: item.time, value: item.signal })));
    histogramSeries.setData(macdData.value.map((item) => ({ time: item.time, value: item.histogram })));

    // ADX 차트 생성
    adxChart = createChart(adxChartContainer.value, {
        width: adxChartContainer.value.clientWidth,
        height: 100,
        layout: { backgroundColor: '#FFFFFF', textColor: '#000' },
        grid: { vertLines: { color: '#e1e3e6' }, horzLines: { color: '#e1e3e6' } },
        timeScale: { borderColor: '#d1d4dc', timeVisible: true }
    });
    adxSeries = adxChart.addLineSeries({ color: 'blue', lineWidth: 1 });
    const pdiSeries = adxChart.addLineSeries({ color: 'green', lineWidth: 1 });
    const mdiSeries = adxChart.addLineSeries({ color: 'red', lineWidth: 1 });

    adxSeries.setData(adxData.value.map((item) => ({ time: item.time, value: item.adx })));
    pdiSeries.setData(adxData.value.map((item) => ({ time: item.time, value: item.pdi })));
    mdiSeries.setData(adxData.value.map((item) => ({ time: item.time, value: item.mdi })));

    synchronizeCrosshair();
    synchronizeTimeScales();

    const resizeObserver = new ResizeObserver(() => {
        candleChart.applyOptions({ width: candleChartContainer.value.clientWidth });
        volumeChart.applyOptions({ width: volumeChartContainer.value.clientWidth });
        supertrendChart.applyOptions({ width: supertrendChartContainer.value.clientWidth });
        rsiChart.applyOptions({ width: rsiChartContainer.value.clientWidth });
        macdChart.applyOptions({ width: macdChartContainer.value.clientWidth });
        adxChart.applyOptions({ width: adxChartContainer.value.clientWidth });
    });
    resizeObserver.observe(candleChartContainer.value);
    resizeObserver.observe(volumeChartContainer.value);
    resizeObserver.observe(supertrendChartContainer.value);
    resizeObserver.observe(rsiChartContainer.value);
    resizeObserver.observe(macdChartContainer.value);
    resizeObserver.observe(adxChartContainer.value);

    onUnmounted(() => {
        resizeObserver.disconnect();
        candleChart.remove();
        volumeChart.remove();
        supertrendChart.remove();
        rsiChart.remove();
        macdChart.remove();
        adxChart.remove();
    });
};

// 시간 동기화
const synchronizeTimeScales = () => {
    const mainTimeScale = candleChart.timeScale();
    const volumeTimeScale = volumeChart.timeScale();
    const supertrendTimeScale = supertrendChart.timeScale();
    const rsiTimeScale = rsiChart.timeScale();
    const macdTimeScale = macdChart.timeScale();
    const adxTimeScale = adxChart.timeScale();

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
    syncTimeScale(mainTimeScale, supertrendTimeScale);
    syncTimeScale(mainTimeScale, rsiTimeScale);
    syncTimeScale(mainTimeScale, macdTimeScale);
    syncTimeScale(mainTimeScale, adxTimeScale);
};

// 수직 지시선 동기화
const synchronizeCrosshair = () => {
    const handleCrosshairMove = (param) => {
        if (param && param.time) {
            const currentTime = param.time;
            const candlestick = data.value.find((item) => item.time === currentTime) || {};
            const volume = volumeData.value.find((item) => item.time === currentTime) || {};
            const supertrend = supertrendData.value.find((item) => item.time === currentTime) || {};
            const rsi = rsiData.value.find((item) => item.time === currentTime) || {};
            const macd = macdData.value.find((item) => item.time === currentTime) || {};
            const adx = adxData.value.find((item) => item.time === currentTime) || {};

            displayData.value = `${selectedDataFile.value} | Open: ${candlestick.open || 0} | High: ${candlestick.high || 0} | Low: ${candlestick.low || 0} | Close: ${candlestick.close || 0} | Volume: ${volume.value || 0}`;
            displayData.value += ` | Supertrend: ${supertrend.value || 0} | RSI: ${rsi.value || 0}`;
            displayData.value += ` | MACD: ${macd.macd?.toFixed(2) || 0} | Signal: ${macd.signal?.toFixed(2) || 0} | Histogram: ${macd.histogram?.toFixed(2) || 0}`;
            displayData.value += ` | ADX: ${adx.adx || 0} | PDI: ${adx.pdi || 0} | MDI: ${adx.mdi || 0}`;
        }
    };

    candleChart.subscribeCrosshairMove(handleCrosshairMove);
    volumeChart.subscribeCrosshairMove(handleCrosshairMove);
    supertrendChart.subscribeCrosshairMove(handleCrosshairMove);
    rsiChart.subscribeCrosshairMove(handleCrosshairMove);
    macdChart.subscribeCrosshairMove(handleCrosshairMove);
    adxChart.subscribeCrosshairMove(handleCrosshairMove);
};

// 초기 마운트 시 CSV 파일 로드
onMounted(() => {
    loadCSV();
});
</script>

<style>
/* 필요한 스타일 추가 */
</style>

