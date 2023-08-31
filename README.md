# frontend-template

React 前端範本，未來新啟動之 project，可按照以下步驟設定新 project。由於各種套件會不斷有新版本，而此範本不一定會一直維護到最新，所以請安照步驟一一進行，不要用 fork 的方式。

## New Project

### 建立新 project
語言選用 typescript

```shell
npx create-react-app my-app --template typescript
```

### 安裝 Dependencies

```shell
# 讓可能因 OS 而有差異的指令在不同 OS 皆可執行
npm install -D cross-env

# 刪除檔案指令
npm install -D del-cli

# 版本控制工具
npm install -D husky

# 讓 npm 指令可以串聯執行，提高可讀性
npm install -D npm-run-all

# 程式碼風格管理工具
npm install -D eslint eslint-import-resolver-typescript eslint-plugin-import eslint-plugin-react prettier
# @typescript-eslint/eslint-plugin @typescript-eslint/parser 這兩個套件已內建於 react template

# tailwind css
npm install -D tailwindcss
```

安裝時附 `-D` 之套件將出現在 `devDependencies`。

### 修正 `package.json`

將 `dependencies` 的 `@testing-library/**`, `@types/**`, `typescript`, `react-scripts` 移至 `devDependencies` 中，因為這些套件都不會在 runtime 執行。

移除 `eslintConfig`，此為程式碼風格的設定，後面會有獨立的設定檔

替換 `scripts`，把範本中的 `scripts` copy & paste 到新 project 中

新增 `engines`，限制 `node` 和 `npm` 版本，這裡不要直接 copy & paste，先 survey 一下當前流行版本

### tsconfig 設定

直接 copy 範本之 `tsconfig.json` 即可。

### husky 設定

```
npm run prepare
npx husky add .husky/pre-commit "npm run pre:commit"
```
讓 husky 在 commit 時跑執行 `pre:commit`，在檢查通過之後才會進行 commit

### eslint 與 prettier 設定

新增檔案 `.prettierrc`、`eslint.json`。

### tailwind css 設定

新增檔案 `tailwind.config.js`
置換檔案 `index.css`


### 移除 cra 建立之檔案

部分由 cra 在建立專案時產生的檔案皆可移除，請參考 template。

## Folder structure

主要參考 Angular 的設計架構 (或是 MVC?)，原則上就是讓分工明確、避免又大又冗長的單一檔案或函數。

### `index.tsx`

整體網站需要的設定如 Router、Redux store。

### `App.tsx`

Layout (navbar、body、snackbar、...)，但若 Layout 隨著 Route 而有根本上的變動，則移至 `Routes.tsx`

### `Routes.tsx`

所有頁面路徑表列於此。

### `page/{pageName}/*`

此資料夾下，一個頁面一個資料夾，資料夾中至少兩個檔案，分別負責需呈現之資料及排版，以 `order` 為例，兩個檔案為 `/page/order/Order.tsx` 及 `/page/order/Order.module.scss`。若此頁面內容繁複，可以拆分成數個元件，置於子資料夾 `component`，各個元件亦需有兩個檔案，如 `/page/order/component/OrderForm.tsx` 及 `/page/order/component/OrderForm.module.scss`

頁面元件不處理任何資料邏輯，只負責「資料呈現」及拋出錯誤時控制 snackbar 等等，此處取得之資料應不須做任何處理即可使用。

### `component/*`

常用的元件如 Button、Iuput 等置於此，亦需有兩個檔案，如 `/component/Navbar.tsx` 及 `/component/Navbar.scss`。原則上所有元件都可以

### `service/*`

負責資料處理，檔名之命名規則為 `{name}Service`，如 `orderService.ts`。API 的呼叫、資料處理、error handling 會於此處進行，回傳元件所需資料或是拋出 human read Error message。

### `util/*`

共用的資料處理工具如 big number 的需求、字串轉換、i18n、http、websocket 等等。

### `redux/*`

以 `redux/store.ts` 為主，定義所有需要的全域 state，切記是存放跨頁面使用的 state，勿濫用，如儲存登入狀態、使用者在 UI 上的操作等。各個 state 以使用功能分類，如 `user`，除了在 `redux/store.ts` 中有定義外，還需要 `redux/userSlice.ts` 來定義儲存的內容。

儲存：原則上在 `**Service.ts` 中呼叫 `dispatch` 進行儲存
使用：在頁面元件中用 `useSelector` 選用需要的資料；在 `**Service.ts` 中則是呼叫 `getState`

### `context/*`

若有需共用但不常更改之參數，會存在 React 提供的 `context` 中，相關設定置於此資料夾，例如顏色主題。

### `model/*`

既然使用了 typescript，就會盡量要求所有參數的 type，所有 object 的 type 皆記錄於此，coding 時使用可以減少許多的 typo。

### `constant/*`

enum 置於此。

### `style/*`

通用的排版參數、設定置於此，關於顏色，為了能讓主題變動的彈性更大，底層 scss 參數分為 `style/_color.scss` 及 `style/_theme.scss`，在頁面元件的 scss 檔直接使用 `style/_theme.scss` 定義的全域變數 `var(--xxx)`，勿直接用 `style/_color.scss`。

## Dependencies

### 全域 state 管理

使用 `redux`，參考：https://redux.js.org/tutorials/quick-start

關於功能類似的 `useContext`，兩者之比較可參考[這篇](https://www.geeksforgeeks.org/whats-the-difference-between-usecontext-and-redux/)跟[這篇](https://dev.to/ruppysuppy/redux-vs-context-api-when-to-use-them-4k3p)。

### UI Design

使用 `Material UI`，只有 `component` 中的元件會使用此 library，頁面元件使用的會是 `component` 中自定義的元件。

### 時間處理

使用 `date-fns`，輕便、較高效能。但 `date-fns` 本身不處理時區，如果需要處理時區的話有另一個 `date-fns-tz`

### Form

使用 `react-hook-form`，其結合 react hooks，開發體驗不錯。

### Coding style 管理

使用 `tslint`、`prettier`、`husky`，tslint、prettier 檢查 code，husky 在 commit 時會自動檢查，確保風格一致。
