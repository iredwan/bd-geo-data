# bd-geo-data

বাংলাদেশের জেলা এবং উপজেলা/থানার বাংলা ডেটা (JSON)। ফ্রি ও ওপেন সোর্স। jsDelivr CDN-এর মাধ্যমে সরাসরি API-এর মতো ব্যবহার করা যায়, কোনো সার্ভার বা API key লাগে না।

## ফাইল

| ফাইল | বিবরণ | এন্ট্রি |
|------|--------|--------|
| `districts_bn_complete.json` | জেলার তালিকা | ৬৪ |
| `upazila_thana.json` | উপজেলা/থানার তালিকা | ৬৩৬ |

## ডেটার ফরম্যাট

দুটি ফাইলেই ডেটা `content` অ্যারের ভেতরে থাকে।

**জেলা**

```json
{ "identifier": "06", "data": "বরিশাল" }
```

**উপজেলা/থানা**

```json
{ "identifier": "0006", "data": "আগৈলঝাড়া", "districtIdentifier": "06" }
```

| ফিল্ড | বিবরণ |
|-------|--------|
| `identifier` | কোড (স্ট্রিং, শুরুর শূন্য ঠিক রাখতে স্ট্রিং হিসেবেই ব্যবহার করুন) |
| `data` | বাংলা নাম |
| `districtIdentifier` | জেলার `identifier`, এর মাধ্যমে জেলার সাথে উপজেলা/থানা মেলে |

## API লিংক

```
https://cdn.jsdelivr.net/gh/iredwan/bd-geo-data@v1.0.0/districts_bn_complete.json
https://cdn.jsdelivr.net/gh/iredwan/bd-geo-data@v1.0.0/upazila_thana.json
```

- CORS সাপোর্ট করে, ব্রাউজার থেকে সরাসরি ফেচ করা যায়।
- প্রোডাকশনে সবসময় ভার্সন ট্যাগ (`@v1.0.0`) ব্যবহার করুন। ডেটা আপডেট হলে নতুন ট্যাগ প্রকাশ করা হবে এবং পুরনো লিংক অপরিবর্তিত থাকবে।
- সর্বশেষ ডেটা পেতে `@main` ব্যবহার করা যায়, তবে ক্যাশের কারণে আপডেট আসতে দেরি হতে পারে।

## ব্যবহারের উদাহরণ

### ১. JavaScript (fetch)

```javascript
const BASE = "https://cdn.jsdelivr.net/gh/iredwan/bd-geo-data@v1.0.0";

async function loadData() {
  const [d, u] = await Promise.all([
    fetch(`${BASE}/districts_bn_complete.json`).then(r => r.json()),
    fetch(`${BASE}/upazila_thana.json`).then(r => r.json()),
  ]);
  return { districts: d.content, upazilas: u.content };
}

// জেলার আইডি দিয়ে উপজেলা/থানা খোঁজা
const { districts, upazilas } = await loadData();
const barishal = upazilas.filter(x => x.districtIdentifier === "06");

// আইডি থেকে নাম বের করা
const nameOf = id => districts.find(x => x.identifier === id)?.data;
```

### ২. HTML ড্রপডাউন (জেলা সিলেক্ট করলে উপজেলা/থানা আসবে)

```html
<select id="district"><option value="">জেলা বাছাই করুন</option></select>
<select id="upazila"><option value="">উপজেলা/থানা বাছাই করুন</option></select>

<script>
const BASE = "https://cdn.jsdelivr.net/gh/iredwan/bd-geo-data@v1.0.0";
const districtEl = document.getElementById("district");
const upazilaEl = document.getElementById("upazila");
let upazilas = [];

(async () => {
  const d = await fetch(`${BASE}/districts_bn_complete.json`).then(r => r.json());
  const u = await fetch(`${BASE}/upazila_thana.json`).then(r => r.json());
  upazilas = u.content;

  d.content
    .sort((a, b) => a.data.localeCompare(b.data, "bn"))
    .forEach(x => districtEl.add(new Option(x.data, x.identifier)));
})();

districtEl.addEventListener("change", () => {
  upazilaEl.length = 1;
  upazilas
    .filter(x => x.districtIdentifier === districtEl.value)
    .sort((a, b) => a.data.localeCompare(b.data, "bn"))
    .forEach(x => upazilaEl.add(new Option(x.data, x.identifier)));
});
</script>
```

### ৩. React

```jsx
import { useEffect, useState } from "react";

const BASE = "https://cdn.jsdelivr.net/gh/iredwan/bd-geo-data@v1.0.0";

export default function GeoSelect() {
  const [districts, setDistricts] = useState([]);
  const [upazilas, setUpazilas] = useState([]);
  const [district, setDistrict] = useState("");

  useEffect(() => {
    fetch(`${BASE}/districts_bn_complete.json`).then(r => r.json()).then(d => setDistricts(d.content));
    fetch(`${BASE}/upazila_thana.json`).then(r => r.json()).then(d => setUpazilas(d.content));
  }, []);

  const filtered = upazilas.filter(x => x.districtIdentifier === district);

  return (
    <>
      <select value={district} onChange={e => setDistrict(e.target.value)}>
        <option value="">জেলা বাছাই করুন</option>
        {districts.map(d => <option key={d.identifier} value={d.identifier}>{d.data}</option>)}
      </select>
      <select>
        <option value="">উপজেলা/থানা বাছাই করুন</option>
        {filtered.map(u => <option key={u.identifier} value={u.identifier}>{u.data}</option>)}
      </select>
    </>
  );
}
```

### ৪. PHP

```php
<?php
$base = "https://cdn.jsdelivr.net/gh/iredwan/bd-geo-data@v1.0.0";
$upazilas = json_decode(file_get_contents("$base/upazila_thana.json"), true)["content"];

$barishal = array_filter($upazilas, fn($x) => $x["districtIdentifier"] === "06");
```

### ৫. Python

```python
import requests

BASE = "https://cdn.jsdelivr.net/gh/iredwan/bd-geo-data@v1.0.0"
upazilas = requests.get(f"{BASE}/upazila_thana.json").json()["content"]

barishal = [x for x in upazilas if x["districtIdentifier"] == "06"]
```

## পরামর্শ

- ডেটা বড় নয় (কয়েক শত KB), তাই একবার ফেচ করে `localStorage` বা মেমরিতে ক্যাশ করে রাখলে বারবার রিকোয়েস্ট লাগে না।
- একই নামের একাধিক থানা আছে (যেমন "কোতোয়ালি", "বিমানবন্দর"), তাই নাম দিয়ে নয়, সবসময় `identifier` দিয়ে খুঁজুন।
- `identifier` সবসময় স্ট্রিং হিসেবে তুলনা করুন (`"06"`, `6` নয়)।

## পরিবর্তনের নোট

- নতুন নাম: টাঙ্গাইলের `4425` এখন "যমুনা সেতু পূর্ব" এবং সিরাজগঞ্জের `1108` এখন "যমুনা সেতু পশ্চিম" (সরকারি প্রজ্ঞাপনে নাম বদলের পর, এপ্রিল ২০২৫)।
- `4396` (ডুপ্লিকেট এন্ট্রি) বাদ দেওয়া হয়েছে।

## লাইসেন্স

রিপোর `LICENSE` ফাইল দেখুন।
