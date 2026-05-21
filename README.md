# โจทย์ฝึก Next.js Clean Architecture FE: Weather Dashboard

## บริบทของโจทย์

โปรเจกต์นี้เป็นการฝึกฝั่ง Frontend เท่านั้น  
ผู้เรียนมีพื้นฐาน Backend อยู่แล้ว แต่ต้องการฝึกการออกแบบโครงสร้าง Frontend ให้เป็นระบบ โดยใช้แนวคิด Clean Architecture

สิ่งที่ต้องฝึกในโจทย์นี้คือ

- การแยกโค้ดเป็น Layer
- การเชื่อมต่อ API ภายนอกด้วย `fetch`
- การแปลงข้อมูลจาก DTO เป็น Entity
- การเขียน business logic แยกออกจาก UI
- การจัดการ loading, error และ empty state
- การใช้ DaisyUI ทำหน้าตา
- การใช้ Motion ทำ animation
- การทำ Component ให้รับ props และแสดงผลเท่านั้น

---

# โจทย์: Weather Dashboard

ให้สร้างหน้าเว็บ Weather Dashboard สำหรับดูสภาพอากาศของเมืองที่เลือก โดยใช้ Next.js App Router และ Clean Architecture

หน้าเว็บต้องอยู่ที่ route

```txt
/weather
```

ผู้ใช้สามารถเลือกเมือง หรือกดปุ่ม refresh เพื่อดึงข้อมูลอากาศล่าสุดจาก API ภายนอกได้

---

# API ที่ใช้

ให้ใช้ Open-Meteo API เพราะไม่ต้องใช้ API Key และเหมาะสำหรับการฝึก Frontend

ตัวอย่าง API สำหรับกรุงเทพฯ

```txt
https://api.open-meteo.com/v1/forecast?latitude=13.7563&longitude=100.5018&current=temperature_2m,relative_humidity_2m,apparent_temperature,weather_code,wind_speed_10m,precipitation&hourly=temperature_2m,precipitation_probability,weather_code&timezone=Asia%2FBangkok
```

หมายเหตุ:

- API นี้เป็น API ภายนอกจริง
- ให้ใช้ `fetch`
- ห้ามใช้ mock data เป็นหลัก
- สามารถมี fallback mock data ได้เฉพาะตอน error หรือ development เท่านั้น

---

# โครงสร้างโปรเจกต์ที่ต้องทำ

```txt
src/
├── app/
│   ├── weather/
│   │   └── page.tsx
│   └── layout.tsx
├── features/
│   └── weather/
│       ├── domain/
│       │   ├── weather.entity.ts
│       │   └── weather.logic.ts
│       ├── data/
│       │   ├── weather.api.ts
│       │   └── weather.repo.ts
│       ├── presenter/
│       │   └── useWeather.ts
│       └── view/
│           ├── WeatherDashboard.tsx
│           ├── CitySelector.tsx
│           ├── CurrentWeatherCard.tsx
│           ├── WeatherStatusBadge.tsx
│           ├── WeatherAdviceCard.tsx
│           ├── WeatherMetricCard.tsx
│           └── HourlyForecastList.tsx
```

---

# Layer ที่ต้องเข้าใจ

## 1. app layer

ไฟล์ใน `src/app` มีหน้าที่กำหนด route เท่านั้น

ตัวอย่าง

```tsx
import { WeatherDashboard } from "@/features/weather/view/WeatherDashboard";

export default function WeatherPage() {
  return <WeatherDashboard />;
}
```

ข้อห้าม:

- ห้าม fetch API ใน `page.tsx`
- ห้ามเขียน logic คำนวณใน `page.tsx`
- ห้ามจัดการ state หลักใน `page.tsx`

---

## 2. domain layer

เก็บ Type และ Logic หลักของระบบ

ไฟล์ที่ต้องมี

```txt
features/weather/domain/weather.entity.ts
features/weather/domain/weather.logic.ts
```

### weather.entity.ts

ให้สร้าง Entity ที่ UI ต้องใช้จริง

```ts
export type WeatherCondition =
  | "clear"
  | "cloudy"
  | "rain"
  | "storm"
  | "unknown";

export type WeatherRiskLevel = "safe" | "warning" | "danger";

export type CurrentWeather = {
  cityName: string;
  temperatureCelsius: number;
  apparentTemperatureCelsius: number;
  humidityPercent: number;
  windSpeedKmh: number;
  precipitationMm: number;
  condition: WeatherCondition;
  riskLevel: WeatherRiskLevel;
  advice: string;
  updatedAt: string;
};

export type HourlyForecast = {
  time: string;
  temperatureCelsius: number;
  rainChancePercent: number;
  condition: WeatherCondition;
};

export type WeatherDashboardData = {
  current: CurrentWeather;
  hourly: HourlyForecast[];
};
```

### weather.logic.ts

ให้เขียน pure function เท่านั้น

ตัวอย่าง function ที่ต้องมี

```ts
export function mapWeatherCodeToCondition(code: number): WeatherCondition;
```

```ts
export function calculateWeatherRisk(params: {
  temperatureCelsius: number;
  humidityPercent: number;
  windSpeedKmh: number;
  precipitationMm: number;
}): WeatherRiskLevel;
```

```ts
export function buildWeatherAdvice(params: {
  condition: WeatherCondition;
  riskLevel: WeatherRiskLevel;
  temperatureCelsius: number;
  rainChancePercent?: number;
}): string;
```

```ts
export function formatTemperature(value: number): string;
```

ตัวอย่างเงื่อนไข

```txt
ถ้าอุณหภูมิ >= 38 = danger
ถ้าอุณหภูมิ >= 34 = warning
ถ้าฝนตกหนัก หรือ precipitation มากกว่า 10 mm = danger
ถ้าลมแรงมากกว่า 40 km/h = warning
นอกนั้น = safe
```

ตัวอย่างคำแนะนำ

```txt
clear + safe      → อากาศดี เหมาะกับกิจกรรมกลางแจ้ง
rain + warning    → มีโอกาสฝนตก ควรพกร่ม
storm + danger    → สภาพอากาศเสี่ยง ควรหลีกเลี่ยงการเดินทาง
temperature สูง  → ควรดื่มน้ำและหลีกเลี่ยงแดดจัด
```

---

## 3. data layer

รับผิดชอบการคุยกับ API และแปลงข้อมูล

ไฟล์ที่ต้องมี

```txt
features/weather/data/weather.api.ts
features/weather/data/weather.repo.ts
```

### weather.api.ts

หน้าที่:

- ยิง API ด้วย `fetch`
- รับข้อมูลดิบจาก API
- ไม่แปลงข้อมูลเป็น Entity
- ไม่เขียน business logic

ตัวอย่าง

```ts
export async function fetchWeatherDto(params: {
  latitude: number;
  longitude: number;
  timezone: string;
}) {
  const url = new URL("https://api.open-meteo.com/v1/forecast");

  url.searchParams.set("latitude", String(params.latitude));
  url.searchParams.set("longitude", String(params.longitude));
  url.searchParams.set(
    "current",
    [
      "temperature_2m",
      "relative_humidity_2m",
      "apparent_temperature",
      "weather_code",
      "wind_speed_10m",
      "precipitation",
    ].join(","),
  );
  url.searchParams.set(
    "hourly",
    ["temperature_2m", "precipitation_probability", "weather_code"].join(","),
  );
  url.searchParams.set("timezone", params.timezone);

  const response = await fetch(url.toString(), {
    cache: "no-store",
  });

  if (!response.ok) {
    throw new Error("Failed to fetch weather data");
  }

  return response.json();
}
```

---

### weather.repo.ts

หน้าที่:

- เรียก `fetchWeatherDto`
- แปลง DTO จาก API เป็น Entity
- เรียกใช้ logic จาก domain
- ซ่อนโครงสร้าง API ไม่ให้ view รู้จัก

ตัวอย่างหน้าที่ของ repo

```txt
API ส่งมา:
current.temperature_2m
current.relative_humidity_2m
current.weather_code

Frontend ต้องใช้:
temperatureCelsius
humidityPercent
condition
advice
riskLevel
```

ข้อสำคัญ:

View ห้ามรู้จักชื่อ field จาก API เช่น

```ts
temperature_2m;
relative_humidity_2m;
weather_code;
```

ชื่อเหล่านี้ควรอยู่ใน data layer เท่านั้น

---

## 4. presenter layer

ไฟล์ที่ต้องมี

```txt
features/weather/presenter/useWeather.ts
```

หน้าที่:

- จัดการ state
- เรียก repository
- จัดการ loading
- จัดการ error
- จัดการเมืองที่เลือก
- ทำปุ่ม refresh
- ดึงข้อมูลใหม่เมื่อเปลี่ยนเมือง

ตัวอย่างค่าที่ hook ควร return

```ts
const { weather, selectedCity, cities, isLoading, error, selectCity, refetch } =
  useWeather();
```

ข้อห้าม:

- ห้ามให้ component view ไปเรียก repo เอง
- ห้ามให้ view fetch API เอง
- ห้ามให้ view แปลง DTO เอง

---

## 5. view layer

ไฟล์ที่ต้องมี

```txt
features/weather/view/WeatherDashboard.tsx
features/weather/view/CitySelector.tsx
features/weather/view/CurrentWeatherCard.tsx
features/weather/view/WeatherStatusBadge.tsx
features/weather/view/WeatherAdviceCard.tsx
features/weather/view/WeatherMetricCard.tsx
features/weather/view/HourlyForecastList.tsx
```

หน้าที่:

- แสดงผล UI
- รับ props
- เรียก hook จาก presenter ได้เฉพาะ component หลัก เช่น `WeatherDashboard`
- component ย่อยควรเป็น presentational component
- ใช้ DaisyUI สำหรับ UI
- ใช้ Motion สำหรับ animation

ตัวอย่าง UI ที่ต้องมี

```txt
1. Header
   - ชื่อหน้า Weather Dashboard
   - ปุ่ม Refresh

2. City Selector
   - เลือกเมือง เช่น Bangkok, Chiang Mai, Phuket, Tokyo, London

3. Current Weather Card
   - แสดงอุณหภูมิปัจจุบัน
   - แสดง feels like / apparent temperature
   - แสดง condition เช่น clear, cloudy, rain

4. Metric Cards
   - humidity
   - wind speed
   - precipitation
   - updated time

5. Weather Advice Card
   - แสดงคำแนะนำจาก domain logic

6. Risk Badge
   - safe
   - warning
   - danger

7. Hourly Forecast
   - แสดง forecast 12 ชั่วโมงถัดไป
   - แสดงเวลา
   - อุณหภูมิ
   - โอกาสฝนตก
   - condition
```

---

# UI Requirement

ให้ใช้ DaisyUI อย่างน้อย:

```txt
- card
- badge
- button
- select
- alert
- loading
- stats หรือ progress
```

ให้ใช้ Motion อย่างน้อย:

```txt
- animate ตอน card ปรากฏ
- animate ตอนเปลี่ยนเมือง
- animate ตอน refresh ข้อมูล
- animate list ของ hourly forecast
```

ตัวอย่างแนวทาง

```tsx
<motion.div
  initial={{ opacity: 0, y: 12 }}
  animate={{ opacity: 1, y: 0 }}
  transition={{ duration: 0.25 }}
>
  ...
</motion.div>
```

---

# เมืองที่ต้องรองรับ

ให้มีอย่างน้อย 5 เมือง

```ts
export const WEATHER_CITIES = [
  {
    id: "bangkok",
    name: "Bangkok",
    latitude: 13.7563,
    longitude: 100.5018,
    timezone: "Asia/Bangkok",
  },
  {
    id: "chiang-mai",
    name: "Chiang Mai",
    latitude: 18.7883,
    longitude: 98.9853,
    timezone: "Asia/Bangkok",
  },
  {
    id: "phuket",
    name: "Phuket",
    latitude: 7.8804,
    longitude: 98.3923,
    timezone: "Asia/Bangkok",
  },
  {
    id: "tokyo",
    name: "Tokyo",
    latitude: 35.6762,
    longitude: 139.6503,
    timezone: "Asia/Tokyo",
  },
  {
    id: "london",
    name: "London",
    latitude: 51.5072,
    longitude: -0.1276,
    timezone: "Europe/London",
  },
];
```

---

# เงื่อนไข Clean Architecture

## ห้ามทำแบบนี้

### 1. ห้าม fetch ใน view

```tsx
// ผิด
useEffect(() => {
  fetch("https://api.open-meteo.com/...");
}, []);
```

### 2. ห้ามเขียน business logic ใน component

```tsx
// ผิด
const risk = temperature > 38 ? "danger" : "safe";
```

### 3. ห้ามให้ view รู้จัก DTO

```tsx
// ผิด
<p>{weather.current.temperature_2m}</p>
```

### 4. ห้ามให้ page.tsx ทำงานเยอะ

```tsx
// ผิด
export default async function Page() {
  const data = await fetch(...);
  return <div>{data.current.temperature_2m}</div>;
}
```

---

# สิ่งที่ต้องส่ง

ผู้เรียนต้องทำให้ครบตามนี้

```txt
1. หน้า /weather ใช้งานได้
2. ใช้ Next.js App Router
3. แยกโครงสร้างตาม Clean Architecture
4. ต่อ API ภายนอกด้วย fetch
5. ใช้ Open-Meteo API
6. แปลง DTO เป็น Entity ผ่าน repository
7. มี domain logic แยกจาก UI
8. มี loading state
9. มี error state
10. มี empty state ถ้าไม่มีข้อมูล forecast
11. มีปุ่ม refresh
12. เลือกเมืองได้อย่างน้อย 5 เมือง
13. ใช้ DaisyUI
14. ใช้ Motion animation
15. View ไม่มี business logic สำคัญ
```

---

# Acceptance Criteria

ถือว่าทำผ่านเมื่อ:

```txt
- เปิด /weather แล้วเห็น Weather Dashboard
- เปลี่ยนเมืองแล้วข้อมูลเปลี่ยนตามเมือง
- กด Refresh แล้วดึงข้อมูลใหม่
- ถ้า API error ต้องแสดง error alert
- ตอนโหลดต้องแสดง loading
- UI ไม่พังตอนข้อมูลบางส่วนไม่มี
- view ไม่เรียก fetch โดยตรง
- view ไม่ใช้ field แบบ DTO จาก API
- logic คำนวณอยู่ใน domain
- repo เป็นตัวแปลง DTO เป็น Entity
```

---

# Bonus Challenge

ถ้าทำโจทย์หลักเสร็จแล้ว ให้ทำ bonus เพิ่ม

## Bonus 1: Auto Refresh

ให้ระบบดึงข้อมูลใหม่อัตโนมัติทุก 5 นาที

เงื่อนไข:

```txt
- ต้อง clear interval ตอน component unmount
- ห้าม refresh ถ้ายัง loading อยู่
- ต้องไม่ยิง API ซ้ำรัว ๆ
```

---

## Bonus 2: Search City

เพิ่มช่องค้นหาเมืองเอง

ตัวอย่าง:

```txt
ผู้ใช้พิมพ์ Bangkok
ระบบค้นหา latitude / longitude
จากนั้นดึง weather ของเมืองนั้น
```

แนวทาง:

ใช้ Open-Meteo Geocoding API หรือทำ city list เพิ่มเองก็ได้

---

## Bonus 3: Favorite Cities

เพิ่มระบบเมืองโปรด

```txt
- กดเพิ่มเมืองโปรดได้
- แสดง list เมืองโปรด
- กดเมืองโปรดเพื่อเปลี่ยนเมืองได้
- เก็บใน localStorage
```

ข้อควรระวัง:

```txt
localStorage ใช้ได้เฉพาะฝั่ง client
```

---

## Bonus 4: Unit Test Domain Logic

เพิ่ม test ให้ไฟล์

```txt
weather.logic.ts
```

ควร test อย่างน้อย:

```txt
- mapWeatherCodeToCondition
- calculateWeatherRisk
- buildWeatherAdvice
- formatTemperature
```

ตัวอย่าง test case:

```txt
temperature 39 → danger
temperature 35 → warning
rain condition → advice ต้องบอกให้พกร่ม
weather_code ฝน → condition เป็น rain
```

---

## Bonus 5: Forecast Chart

เพิ่มกราฟอุณหภูมิ 12 ชั่วโมงถัดไป

ข้อมูลที่ใช้:

```txt
hourly temperature
hourly time
```

จะใช้ library เช่น Recharts ได้

สิ่งที่ต้องแสดง:

```txt
- เส้นอุณหภูมิ
- เวลา
- tooltip
```

---

## Bonus 6: Theme Toggle

เพิ่มปุ่มเปลี่ยน theme ของ DaisyUI

```txt
light
dark
cupcake
business
```

ให้จำ theme ล่าสุดด้วย localStorage

---

## Bonus 7: Better Error Handling

แยก error เป็นหลายกรณี

```txt
- network error
- API response error
- city not found
- invalid data shape
```

แล้วแสดงข้อความให้ผู้ใช้เข้าใจง่าย

---

# Setup Command

สร้างโปรเจกต์

```bash
npx create-next-app@latest weather-clean-fe
```

ตัวเลือกที่แนะนำ

```txt
TypeScript: Yes
ESLint: Yes
Tailwind CSS: Yes
App Router: Yes
src directory: Yes
Import alias: Yes
```

ติดตั้ง package

```bash
npm install daisyui motion
```

ถ้าทำ bonus chart

```bash
npm install recharts
```

ถ้าทำ unit test

```bash
npm install -D vitest
```

---

# สรุปโจทย์แบบสั้น

ให้สร้าง Weather Dashboard ด้วย Next.js โดยใช้ Clean Architecture ฝั่ง Frontend

เป้าหมายไม่ใช่แค่ทำหน้าเว็บให้แสดงข้อมูลได้ แต่ต้องฝึกแยกความรับผิดชอบของโค้ดให้ชัดเจน

```txt
app        → route เท่านั้น
view       → แสดงผลเท่านั้น
presenter  → state และ interaction
data       → fetch และ map DTO
domain     → entity และ business logic
```

โจทย์นี้เหมาะสำหรับคนที่มีพื้นฐาน Backend แล้ว และอยากฝึกเขียน Frontend ให้เป็นระบบมากขึ้น
