---
layout: slide
title: "Katlanır Bomlu Vinç: İleri Seviye Tasarım"
---

# Katlanır bomlu vinç tasarımını geliştirelim! 🏗️

## 1) Tasarım girdileri
- **Maksimum yük:** 2.5 ton
- **Nominal erişim:** 4.0 m
- **Maksimum erişim:** 8.5 m
- **Slew (dönüş):** 360° sürekli
- **Emniyet yaklaşımı:** EN 12999'a uyumlu yük eğrisi mantığı

## 2) Kinematik model (2D ileri kinematik + limit kontrol)

```js
const crane = {
  baseHeight: 1.2,
  segments: [3.6, 2.8, 1.9],
  jointLimitsDeg: [
    { min: -10, max: 75 },
    { min: -95, max: 20 },
    { min: -60, max: 60 }
  ]
};

const degToRad = (d) => (d * Math.PI) / 180;

function clampToLimits(anglesDeg, limits) {
  return anglesDeg.map((a, i) => Math.max(limits[i].min, Math.min(limits[i].max, a)));
}

function forwardKinematics2D(baseX, baseY, lengths, anglesDeg) {
  let x = baseX;
  let y = baseY;
  let theta = 0;

  for (let i = 0; i < lengths.length; i++) {
    theta += degToRad(anglesDeg[i]);
    x += lengths[i] * Math.cos(theta);
    y += lengths[i] * Math.sin(theta);
  }

  return { tipX: x, tipY: y, totalAngleRad: theta };
}

const commandAngles = [42, -38, 12];
const safeAngles = clampToLimits(commandAngles, crane.jointLimitsDeg);
const tip = forwardKinematics2D(0, crane.baseHeight, crane.segments, safeAngles);

console.log({ commandAngles, safeAngles, tip });
```

## 3) Basit kapasite / stabilite kontrolü

```js
function requiredStabilizingMoment(loadTon, outreachM, dynamicFactor = 1.25) {
  return loadTon * outreachM * dynamicFactor; // ton·m (yaklaşık)
}

function isWithinChart(loadTon, outreachM) {
  // Örnek mini yük eğrisi (ton)
  const chart = [
    { reach: 3, maxLoad: 3.2 },
    { reach: 4, maxLoad: 2.5 },
    { reach: 6, maxLoad: 1.6 },
    { reach: 8, maxLoad: 1.0 }
  ];

  const point = chart.find((p) => outreachM <= p.reach) ?? chart[chart.length - 1];
  return loadTon <= point.maxLoad;
}

const loadTon = 2.2;
const outreachM = Math.abs(tip.tipX);
console.log('Gerekli denge momenti (yaklaşık):', requiredStabilizingMoment(loadTon, outreachM));
console.log('Yük eğrisine uygun mu?', isWithinChart(loadTon, outreachM));
```

## 4) Sonraki mühendislik adımları
- Silindir boyu → eklem açısı için **ters kinematik**.
- Yapısal doğrulama için bom kesitlerinde **gerilme / burkulma** kontrolü.
- Emniyet PLC logic'i: limit switch + moment limiter + soft stop.
