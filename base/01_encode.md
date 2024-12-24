# 编程编码基础

## 1. JavaScript 中的二进制转换

### 1.1 **ArrayBuffer → Blob**

`ArrayBuffer` 是一个通用的固定大小的原始二进制缓冲区，而 `Blob` 是表示二进制数据的文件对象。可以将一个 `ArrayBuffer` 转换为 `Blob`，或者从 `ArrayBuffer` 中提取一部分字节并转换成 `Blob`。

```javascript
// 将完整的 ArrayBuffer 转换为 Blob
const buffer = new ArrayBuffer(256);
const blob = new Blob([buffer]);

// 从 ArrayBuffer 中提取一部分字节并转换为 Blob
const blob = new Blob([new Uint8Array(buffer, byteOffset, length)]);
```

### 1.2 **ArrayBuffer → Base64**

`Base64` 是一种用 64 个字符来表示二进制数据的编码方式。JavaScript 中可以将 `ArrayBuffer` 转换为 `Base64` 字符串。

```javascript
const base64 = btoa(
  String.fromCharCode.apply(null, new Uint8Array(arrayBuffer))
);
```

### 1.3 **Base64 → Blob**

可以将 `Base64` 编码的数据转换为 `Blob` 对象。这个过程涉及到将 `Base64` 字符串解码为字节数据，然后再通过 `Blob` 封装成二进制数据。

```javascript
const base64toBlob = (base64Data, contentType, sliceSize) => {
  const byteCharacters = atob(base64Data);
  const byteArrays = [];

  for (let offset = 0; offset < byteCharacters.length; offset += sliceSize) {
    const slice = byteCharacters.slice(offset, offset + sliceSize);
    const byteNumbers = new Array(slice.length);

    for (let i = 0; i < slice.length; i++) {
      byteNumbers[i] = slice.charCodeAt(i);
    }

    const byteArray = new Uint8Array(byteNumbers);
    byteArrays.push(byteArray);
  }

  return new Blob(byteArrays, { type: contentType });
};
```

### 1.4 **Blob → ArrayBuffer**

将 `Blob` 转换为 `ArrayBuffer` 需要使用 `FileReader` 来读取 `Blob` 内容。

```javascript
function blobToArrayBuffer(blob) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = () => resolve(reader.result);
    reader.onerror = () => reject;
    reader.readAsArrayBuffer(blob);
  });
}
```

### 1.5 **Blob → Base64**

将 `Blob` 转换为 `Base64` 数据，可以通过 `FileReader` 读取为数据 URL 格式（即 base64 格式）。

```javascript
function blobToBase64(blob) {
  return new Promise((resolve) => {
    const reader = new FileReader();
    reader.onloadend = () => resolve(reader.result);
    reader.readAsDataURL(blob);
  });
}
```

### 1.6 **Blob → Object URL**

`Blob` 可以通过 `URL.createObjectURL` 转换为一个对象 URL，这种 URL 可以被浏览器用来引用该 `Blob` 对象的内容。

```javascript
const objectUrl = URL.createObjectURL(blob);
```

---

## 2. 认识数字单元

数字的位数决定了它可以表示的数值范围。在 JavaScript 中，常见的数字类型有 `uint8`、`uint16`、`uint32` 等，它们代表了不同的二进制位数，并对应不同的数值范围。

### 2.1 数字位数系列

- **Uint8**：8 位二进制数据，表示数值范围 `0` 到 `255`（即 `2^8 - 1`）。
- **Uint16**：16 位二进制数据，表示数值范围 `0` 到 `65535`（即 `2^16 - 1`）。
- **Uint32**：32 位二进制数据，表示数值范围 `0` 到 `4294967295`（即 `2^32 - 1`）。

这些类型通常会以不同的数组类型（如 `Uint8Array`、`Uint16Array` 等）存在，适应不同精度的数据需求。

---

## 3. 字符编码

### 3.1 **UTF-8 编码**

UTF-8 是一种可变长度字符编码，能够为每个字符使用不同数量的字节。UTF-8 中的字符编码范围从 0 到 255。它采用不同字节数来表示不同范围的字符。常见字符（例如英语字母）使用一个字节表示，而对于某些字符（如汉字）则使用多个字节。

### 3.2 **UTF-16 和 UTF-32 编码**

- **UTF-16**：使用 16 位（2 字节）表示字符，但有一些字符（如某些表情符号）会使用 32 位（4 字节）来表示。
- **UTF-32**：每个字符都用 32 位（4 字节）表示，因此可以表示全球范围内的所有字符，但会占用更多的空间。

### 3.3 **Unicode 编码**

Unicode 是一个字符集，定义了所有书写系统中的字符，并为每个字符分配一个唯一的编号。例如，中文字符 `中` 在 Unicode 中的编号是 `\u4E2D`，这个编号表示字符的唯一标识。Unicode 通过不同的编码方式（如 UTF-8、UTF-16）来表示这些字符。
