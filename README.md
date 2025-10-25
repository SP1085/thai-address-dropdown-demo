# 🇹🇭 Thai Address Dropdown Demo: ระบบเลือกที่อยู่แบบเรียงซ้อน (Static Data)

โปรเจกต์นี้คือชุดโค้ดและคู่มือที่สมบูรณ์สำหรับการสร้าง **Cascading Dropdown** (รายการเลือกแบบเรียงซ้อน) ของข้อมูลที่อยู่ประเทศไทย (จังหวัด → อำเภอ → ตำบล) โดยใช้เทคนิค **Client-Side** ทั้งหมด

เป็นทรัพยากรสำหรับผู้ที่ต้องการเรียนรู้และใช้งาน **Vanilla JavaScript**, **Fetch API**, และการจัดการ DOM ในการจัดการข้อมูลขนาดใหญ่

***

## 🎯 แนวคิดหลักและการนำไปใช้งาน (Core Concepts)

| แนวคิดหลัก | คำอธิบายโดยย่อ | เมธอด/เทคนิคที่ใช้ |
| :--- | :--- | :--- |
| **Asynchronous Data Fetching** | การดึงข้อมูล JSON ขนาดใหญ่จาก API โดยไม่บล็อกการทำงานของ UI | `async`/`await`, `fetch()` |
| **Client-Side Filtering** | การจัดเก็บข้อมูลทั้งหมดไว้ใน Array และใช้ JavaScript กรองข้อมูลชุดย่อยที่เกี่ยวข้องเมื่อผู้ใช้เลือกรายการด้านบน | `Array.prototype.filter()` |
| **Data Preparation** | การรวบรวมข้อมูลสุดท้าย (ID, ชื่อ, รหัสไปรษณีย์) และจัดเตรียมเป็น **JSON String** เพื่อส่งต่อไปบันทึกที่ Backend | `JSON.stringify()` |

***

## 🚀 วิธีการใช้งาน (How to Use)

ทำตามขั้นตอนเหล่านี้เพื่อนำระบบ Dropdown ไปใช้งานในโปรเจกต์ HTML ของคุณ:

### ขั้นตอนที่ 1: เตรียมโครงสร้าง HTML

คัดลอกโครงสร้าง HTML ต่อไปนี้ไปวางในส่วน `<form>` ของคุณ ตรวจสอบให้แน่ใจว่า **`id`** ถูกกำหนดตามที่ระบุไว้:

```html
<div class="form-group">
    <label for="province">จังหวัด:</label>
    <select id="province"><option value="">--</option></select>
</div>
<div class="form-group">
    <label for="district">อำเภอ/เขต:</label>
    <select id="district"><option value="">--</option></select>
</div>
<div class="form-group">
    <label for="subdistrict">ตำบล/แขวง:</label>
    <select id="subdistrict"><option value="">--</option></select>
</div>

<input type="text" id="postid_field" placeholder="รหัสไปรษณีย์" readonly>
```

### ขั้นตอนที่ 2: วางโค้ด JavaScript

คัดลอก โค้ด JavaScript ฉบับเต็ม ที่อยู่ด้านล่างนี้ และนำไปวางในแท็ก <script> ก่อนปิดแท็ก </body> ของไฟล์ HTML ของคุณ:

```JavaScript
<script>
// 💡 Section 1: การกำหนดค่าเริ่มต้นและแหล่งข้อมูล (URLs)
const PROVINCE_URL = '[https://raw.githubusercontent.com/kongvut/thai-province-data/master/api/latest/province.json](https://raw.githubusercontent.com/kongvut/thai-province-data/master/api/latest/province.json)';
const DISTRICT_URL = '[https://raw.githubusercontent.com/kongvut/thai-province-data/master/api/latest/district.json](https://raw.githubusercontent.com/kongvut/thai-province-data/master/api/latest/district.json)';
const SUBDISTRICT_URL = '[https://raw.githubusercontent.com/kongvut/thai-province-data/master/api/latest/sub_district.json](https://raw.githubusercontent.com/kongvut/thai-province-data/master/api/latest/sub_district.json)';

// การอ้างอิง DOM Elements (ID ต้องตรงกับ HTML)
const provinceSelect = document.getElementById('province');
const districtSelect = document.getElementById('district');
const subdistrictSelect = document.getElementById('subdistrict');
const postidField = document.getElementById('postid_field'); // ช่องรหัสไปรษณีย์

let allProvinces = [];
let allDistricts = [];
let allSubdistricts = [];

// 💡 Section 2: Helper Functions

// [Helper Function] ดึงข้อมูลจาก URL ด้วย Fetch API
async function fetchData(url) {
    try {
        const response = await fetch(url);
        return response.ok ? await response.json() : null;
    } catch (error) {
        console.error('Fetch failed:', error);
        return null;
    }
}

// [Helper Function] เติมตัวเลือกให้กับ Select Element
function populateSelect(selectElement, data, defaultText) {
    selectElement.innerHTML = `<option value="">${defaultText}</option>`;
    if (Array.isArray(data) && data.length > 0) {
        data.forEach(item => {
            const option = document.createElement('option');
            option.value = item.id;
            option.textContent = item.name_th;
            selectElement.appendChild(option);
        });
        selectElement.disabled = false;
    } else {
        selectElement.disabled = true;
    }
}

// [Main Function] โหลดข้อมูลทั้งหมดเมื่อ DOM โหลดเสร็จ
async function loadAllData() {
    allProvinces = await fetchData(PROVINCE_URL);
    allDistricts = await fetchData(DISTRICT_URL);
    allSubdistricts = await fetchData(SUBDISTRICT_URL);
    
    if (allProvinces) {
        populateSelect(provinceSelect, allProvinces, '-- เลือกจังหวัด --');
    }
}

// 💡 Section 3: Event Listeners (Cascading Logic)

// 1. เมื่อเลือกจังหวัด
provinceSelect.addEventListener('change', () => {
    const provinceId = parseInt(provinceSelect.value);
    
    districtSelect.disabled = subdistrictSelect.disabled = true;
    districtSelect.innerHTML = '<option value="">กรุณาเลือกจังหวัดก่อน</option>';
    subdistrictSelect.innerHTML = '<option value="">กรุณาเลือกอำเภอ/เขตก่อน</option>';
    postidField.value = ''; 
    
    if (provinceId) {
        // 🚀 กรองอำเภอที่ตรงกับ province_id
        const filteredDistricts = allDistricts.filter(d => d.province_id === provinceId);
        populateSelect(districtSelect, filteredDistricts, '-- เลือกอำเภอ/เขต --');
    }
});

// 2. เมื่อเลือกอำเภอ
districtSelect.addEventListener('change', () => {
    const districtId = parseInt(districtSelect.value);
    subdistrictSelect.disabled = true;
    subdistrictSelect.innerHTML = '<option value="">กรุณาเลือกอำเภอ/เขตก่อน</option>';
    postidField.value = ''; 

    if (districtId) {
        // 🚀 กรองตำบลที่ตรงกับ district_id
        const filteredSubdistricts = allSubdistricts.filter(s => s.district_id === districtId);
        populateSelect(subdistrictSelect, filteredSubdistricts, '-- เลือกตำบล/แขวง --');
    }
});

// 3. เมื่อเลือกตำบล (Final Step: ตั้งค่ารหัสไปรษณีย์และเตรียมข้อมูลส่ง Server)
subdistrictSelect.addEventListener('change', () => {
    const subdistrictId = parseInt(subdistrictSelect.value);
    postidField.value = '';

    if (subdistrictId) {
        // ค้นหา Object ข้อมูลเต็ม
        const selectedSubdistrict = allSubdistricts.find(s => s.id === subdistrictId);
        
        if (selectedSubdistrict) {
            // ตั้งค่ารหัสไปรษณีย์อัตโนมัติ (ใช้งานจริง)
            postidField.value = selectedSubdistrict.zip_code;

            // 💡 ณ จุดนี้ โค้ดของคุณพร้อมที่จะสร้าง JSON Object และส่งไปยัง Backend
            /*
            const selectedProvince = allProvinces.find(p => p.id === parseInt(provinceSelect.value));
            const selectedDistrict = allDistricts.find(d => d.id === parseInt(districtSelect.value));
            
            const dataToSave = { 
               province: { id: selectedProvince.id, name: selectedProvince.name_th },
               district: { id: selectedDistrict.id, name: selectedDistrict.name_th },
               subdistrict: { id: selectedSubdistrict.id, name: selectedSubdistrict.name_th, postid: selectedSubdistrict.zip_code }
            };
            console.log("JSON พร้อมส่ง:", JSON.stringify(dataToSave));
            fetch('/api/save-address', { method: 'POST', body: JSON.stringify(dataToSave) });
            */
        }
    }
});

// 💡 Section 4: เริ่มต้นการทำงาน (Initialization)
// รอให้ DOM โหลดเสร็จก่อนจึงค่อยเรียกฟังก์ชัน loadAllData
document.addEventListener('DOMContentLoaded', loadAllData);
</script>
```

### 📚 แหล่งข้อมูล API (Reference)

ข้อมูล Static JSON ที่ใช้ในโปรเจกต์นี้มาจาก Repository สาธารณะของ คุณ KongVut

| ระดับข้อมูล | URL สำหรับ Fetch API | 
| :--- | :--- |
| จังหวัด (Province) | https://raw.githubusercontent.com/kongvut/thai-province-data/master/api/latest/province.json
| อำเภอ (District) | https://raw.githubusercontent.com/kongvut/thai-province-data/master/api/latest/district.json
| ตำบล (Subdistrict) | https://raw.githubusercontent.com/kongvut/thai-province-data/master/api/latest/sub_district.json

Reference: [kongvut/thai-province-data ](https://github.com/kongvut/thai-province-data.git)
