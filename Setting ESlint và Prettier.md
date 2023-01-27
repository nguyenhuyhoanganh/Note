## Cài extension `ESlint` và `Prettier - Code formatter`

- Cho phép sử dụng `Shift + Alt + F` hoặc save để format code theo `Prettier`

## Setting User, chỉnh sửa:

- Editor: Tab Size = 2
- Editor: Default Forrmatter = Prettier - Code formatter
- Editor: Forrmat On Save = True
- Files: Eol = Auto

## Tạo file `.editorconfig` nội dung:

```bash
[*]
indent_style = space
indent_size = 2
```

## Tạo file `.prettierrc` nội dung:

```json
{
  "semi": false,
  "trailingComma": "none",
  "tabWidth": 2,
  "endOfLine": "auto",
  "useTabs": false,
  "singleQuote": true,
  "printWidth": 120,
  "jsxSingleQuote": true
}
```

- loại bỏ dấu `;`
- loại bỏ dấu `,` của property cuối cùng trong 1 object
- khoảng cách tab là 2
- tự động xuống dòng
- không sử dụng tab, thay tab bằng 2 space
- dùng nháy đơn `'` thay nháy kép
- độ dài 1 hàng là 120
- sử dụng nháy đơn trong jsx

## Cài các `devDependencies` sau:

```bash
npm i prettier eslint-plugin-prettier eslient-config-prettier -D
```

## Tạo file `.eslintrc` nội dung:

```json
{
  "extends": ["react-app", "prettier"],
  "plugins": ["react", "prettier"],
  "rules": {
    "prettier/prettier": [
      "warn",
      {
        "semi": false,
        "trailingComma": "none",
        "tabWidth": 2,
        "endOfLine": "auto",
        "useTabs": false,
        "singleQuote": true,
        "printWidth": 120,
        "jsxSingleQuote": true
      }
    ]
  }
}
```

## Thêm `scripts` vào `package.json`:

```json
{
  "lint": "eslint --ext js,jsx,ts,tsx src/",
  "lint:fix": "eslint --fix --ext js,jsx,ts,tsx src/",
  "prettier": "prettier --check \"src/**/(*.jsx|*.js|*.tsx|*ts|*.css|*.scss)\"",
  "prettier:fix": "prettier --write \"src/**/(*.jsx|*.js|*.tsx|*ts|*.css|*.scss)\""
  // check các file có đuôi tương ứng trong folder src
}
```

### Giúp khai báo các command:

- `npm run lint` : kiểm tra lỗi eslint
- `npm run lint:fix` : tự động fix hầu hết lỗi eslint
- `npm run prettier` : kiểm tra lỗi prettier format
- `npm run prittier:fix` : tự động fix lỗi prettier format

## Thêm các file `.prettierignore` và `.eslintignore` để ignore những file không muốn prettier format và kiểm tra băng eslint
