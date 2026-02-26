# ชั่วโมงที่ 7: Unit test ด้วย Jestjs

## วัตถุประสงค์
- ผู้เรียนสามารถเขียน unit test ด้วย Jestjs ได้
- เข้าใจการใช้ Jest API เบื้องต้น

## เนื้อหา
### 1. Introduction to Unit Testing
- Unit Testing คืออะไร
- ความสำคัญของ Unit Testing

### 2. Setting Up Jest
- ติดตั้ง Jest ในโปรเจค (เช่น `npm install jest --save-dev`)
- Configure package.json และสร้าง script (`"test": "jest"`)

### 3. Writing Your First Test
- โครงสร้างและ syntax ของการเขียน test
- ตัวอย่าง:
```javascript
function sum(a, b) {
  return a + b;
}
module.exports = sum;
```
Test case:
```javascript
const sum = require('./sum');

test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3);
});
```

### 4. Advanced Features in Jest
- Mocking
- Matchers
- Setup & Teardown

### 5. Workshop
- สร้าง unit test สำหรับฟังก์ชันที่ custom โดยผู้เรียนเอง
- ประเมินตัวอย่าง code และเขียน test case เพิ่มเติม

## Workshop Example/Task
### Task
1. ให้ผู้เรียนพัฒนาฟังก์ชัน multiply และทดสอบ
ตัวอย่าง:
```javascript
// multiply.js
function multiply(a, b) {
  return a * b;
}
module.exports = multiply;
```
Test case:
```javascript
const multiply = require('./multiply');

test('multiplies 2 * 3 to equal 6', () => {
  expect(multiply(2, 3)).toBe(6);
});
```

 ให้ผู้เรียน: Code เพลย์แลักจริง เอาจาก Instruction above
