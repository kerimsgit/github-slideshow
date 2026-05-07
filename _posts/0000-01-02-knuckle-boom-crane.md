---
layout: slide
title: "Katlanır Bomlu Vinç Tasarımı"
---

# Katlanır bomlu vinç tasarlayalım! 🏗️

## 1) Tasarım hedefleri
- **Kapasite:** 2.5 ton @ 4 m erişim
- **Maks. erişim:** 8.5 m
- **Dönüş açısı:** 360° sürekli
- **Güvenlik katsayısı:** yapısal minimum 1.7

## 2) Basit geometri modeli (2D)

```js
const design = {
  baseHeight: 1.2,          // m
  boomSegments: [3.6, 2.8, 1.9], // m
  jointLimitsDeg: [
    { min: -10, max: 75 },  // ana bom
    { min: -95, max: 20 },  // dirsek
    { min: -60, max: 60 }   // jib
  ]
};

function tipPosition2D(baseX, baseY, lengths, anglesDeg) {
  const toRad = d => d * Math.PI / 180;
  let x = baseX;
  let y = baseY;
  let theta = 0;

  for (let i = 0; i < lengths.length; i++) {
    theta += toRad(anglesDeg[i]);
    x += lengths[i] * Math.cos(theta);
    y += lengths[i] * Math.sin(theta);
  }

  return { x, y };
}

const tip = tipPosition2D(0, design.baseHeight, design.boomSegments, [42, -38, 12]);
console.log('Tip koordinatı:', tip);
```

## 3) Stabilite kontrolü (hızlı kontrol)
- Yük momenti: `M_yük = Yük * YatayUzaklık`
- Denge kriteri: `M_devirmeye karşı >= 1.25 * M_yük`
- Operasyonda ayaklar (outrigger) **tam açık** varsayılır.

## 4) Sonraki adım
Hidrolik silindir boylarından eklem açılarını bulan ters kinematik + yük diyagramı hesaplayıcısı.
