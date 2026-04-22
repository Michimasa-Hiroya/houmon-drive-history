# サイドバー タイムライン強化型 UI改善 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 訪問先編集セクションを「タイムライン強化型」にリデザインし、初心者でも迷わず操作できるUIにする。

**Architecture:** `index.html` 単一ファイルの `<style>` と JSX マークアップを変更。機能ロジックは一切触らない。変更は4フェーズ：CSS追加 → state追加 → 出発/帰社カード → 訪問先ループ。

**Tech Stack:** React 18 (UMD/Babel), Tailwind CSS CDN, 単一 HTML ファイル

---

## ファイル構成

- Modify: `index.html:10-132` — `<style>` セクション（CSS クラス追加）
- Modify: `index.html:1408` — state宣言（`insertOpenAt` 追加）
- Modify: `index.html:2129-2288` — サイドバーJSXのスクロールリスト全体

---

## Task 1: CSS クラスを追加する

**Files:**
- Modify: `index.html` — `<style>` ブロック末尾（`</style>` の直前、約131行目）

- [ ] **Step 1: 以下のCSSを `</style>` タグの直前に挿入する**

```css
  /* ── タイムライン ── */
  .tl-wrap { position: relative; padding-left: 26px; }
  .tl-wrap::before {
    content: ''; position: absolute; left: 10px; top: 16px; bottom: 16px;
    width: 2px; background: linear-gradient(to bottom, #10b981 0%, #3b82f6 30%, #64748b 95%);
  }
  .tl-node { position: relative; margin-bottom: 4px; }
  .tl-dot {
    position: absolute; left: -21px; top: 10px;
    width: 22px; height: 22px; border-radius: 50%;
    display: flex; align-items: center; justify-content: center;
    font-size: 9px; font-weight: 700; color: white; z-index: 1;
    box-shadow: 0 0 0 3px white;
  }

  /* ── 挿入ボタン（大型） ── */
  .tl-insert-btn {
    width: 100%; padding: 7px 12px; margin: 5px 0;
    background: #eff6ff; border: 1.5px dashed #93c5fd; border-radius: 10px;
    font-size: 12px; font-weight: 600; color: #3b82f6;
    cursor: pointer; display: flex; align-items: center; justify-content: center; gap: 6px;
    transition: background 0.12s;
  }
  .tl-insert-btn:hover { background: #dbeafe; border-color: #3b82f6; }
  .tl-insert-btn:active { background: #bfdbfe; }
  .tl-insert-choices { display: flex; gap: 8px; margin: 4px 0 6px; }
  .tl-insert-choices button {
    flex: 1; padding: 7px 0; border-radius: 10px; border: 1.5px solid; cursor: pointer;
    font-size: 12px; font-weight: 600; text-align: center;
  }

  /* ── ドラッグハンドル（枠付き） ── */
  .drag-box {
    display: flex; flex-direction: column; gap: 3px;
    cursor: grab; padding: 3px 5px; border-radius: 6px;
    background: #f8fafc; border: 1px solid #e2e8f0; flex-shrink: 0;
    touch-none: none; user-select: none;
  }
  .drag-box:active { cursor: grabbing; background: #f1f5f9; }
  .drag-box span { display: block; width: 14px; height: 2px; background: #94a3b8; border-radius: 1px; }

  /* ── アクションボタン（ラベル付き） ── */
  .act-btn {
    font-size: 11px; padding: 5px 9px; border-radius: 8px;
    border: 1px solid; cursor: pointer; font-weight: 600; white-space: nowrap;
  }
  .act-btn-fav  { background: #fffbeb; color: #92400e; border-color: #fde68a; }
  .act-btn-del  { background: #fef2f2; color: #b91c1c; border-color: #fca5a5; }
  .act-btn-map  { background: #f0fdf4; color: #059669; border-color: #6ee7b7; }
  .act-btn-gray { background: #f8fafc; color: #475569; border-color: #cbd5e1; }
```

- [ ] **Step 2: ブラウザでページを開き、コンソールエラーがないことを確認する**

`index.html` をブラウザで開く（またはリロード）。DevTools Console にエラーが出ないこと。

- [ ] **Step 3: コミット**

```bash
git add index.html
git commit -m "style: add timeline, insert-btn, drag-box, act-btn CSS classes"
```

---

## Task 2: `insertOpenAt` state を追加する

**Files:**
- Modify: `index.html:1408` — state宣言ブロック

- [ ] **Step 1: 以下の1行を `const [showMapPopup, setShowMapPopup] = useState(false);` の直後に追加する**

変更前（1408行目付近）:
```js
  const [showMapPopup, setShowMapPopup] = useState(false); // 'insert' | 'add' | false
  const [showPreview, setShowPreview] = useState(false);
```

変更後:
```js
  const [showMapPopup, setShowMapPopup] = useState(false); // 'insert' | 'add' | false
  const [insertOpenAt, setInsertOpenAt] = useState(null); // null | 'base' | 数値(stop index)
  const [showPreview, setShowPreview] = useState(false);
```

- [ ] **Step 2: ブラウザでリロードしコンソールエラーがないことを確認**

- [ ] **Step 3: コミット**

```bash
git add index.html
git commit -m "feat: add insertOpenAt state for expandable insert buttons"
```

---

## Task 3: スクロールリストをタイムラインラッパーで囲み、出発・帰社カードを改善する

**Files:**
- Modify: `index.html:2129-2290` — スクロールリスト全体

このタスクでは以下を変更する：
1. `<div className="flex-1 overflow-y-auto sidebar-scroll px-3 py-2">` の直下に `<div className="tl-wrap">` を追加
2. 出発カード全体を `.tl-node` で包み、`.tl-dot` を追加
3. 帰社カード全体を `.tl-node` で包み、`.tl-dot` を追加
4. 出発・帰社の ☆/📍 ボタンをラベル付きに変更
5. リスト末尾（帰社カードの閉じ `</div>` の後）に `</div>` （tl-wrap閉じ）を追加

- [ ] **Step 1: 出発カード周辺のJSXを以下に置き換える**

変更前（約2131〜2145行）:
```jsx
            {/* 事業所 */}
            <div className="flex items-center gap-2 bg-emerald-50 border border-emerald-100 rounded-xl px-3 py-2.5 mb-2">
              <div className="w-8 h-8 bg-emerald-600 rounded-full flex items-center justify-center text-white text-xs font-bold flex-shrink-0">発</div>
              <div className="flex-1 min-w-0">
                <div className="text-xs font-semibold text-emerald-700">出発（{rec.base?.districtName}）</div>
                <div className="text-xs text-slate-400 truncate">{rec.base?.address}</div>
              </div>
              <div className="flex flex-shrink-0 gap-0.5">
                <button onClick={()=>setFavMemoModal(rec.base)} title="お気に入り登録"
                  className="w-8 h-8 flex items-center justify-center text-slate-300 hover:text-yellow-400 active:text-yellow-500 text-lg">☆</button>
                <button onClick={()=>setMapMode("start")} title="地図から選択"
                  className={`w-8 h-8 flex items-center justify-center rounded-lg text-sm ${mapMode==="start"?"bg-emerald-200 text-emerald-800":"text-emerald-600 hover:bg-emerald-200"}`}>📍</button>
                <button onClick={()=>setShowBaseEdit("start")} title="検索して変更"
                  className="w-8 h-8 flex items-center justify-center text-slate-400 hover:text-emerald-600 text-sm">🔍</button>
              </div>
            </div>
```

変更後:
```jsx
            {/* ── タイムラインラッパー ── */}
            <div className="tl-wrap">

            {/* 事業所（出発） */}
            <div className="tl-node">
              <div className="tl-dot" style={{background:'#10b981'}}>発</div>
              <div className="flex items-center gap-2 bg-emerald-50 border border-emerald-200 rounded-xl px-3 py-2.5 mb-1">
                <div className="flex-1 min-w-0">
                  <div className="text-xs font-semibold text-emerald-700">出発（{rec.base?.districtName}）</div>
                  <div className="text-xs text-slate-400 truncate">{rec.base?.address}</div>
                </div>
                <div className="flex flex-shrink-0 gap-1.5">
                  <button onClick={()=>setFavMemoModal(rec.base)} className="act-btn act-btn-fav">★ 登録</button>
                  <button onClick={()=>setMapMode("start")} className={`act-btn ${mapMode==="start"?"act-btn-map":"act-btn-gray"}`}>📍 地図</button>
                </div>
              </div>
            </div>
```

- [ ] **Step 2: 帰社カード周辺のJSXを以下に置き換える**

変更前（約2272〜2288行）:
```jsx
            {/* 帰社 */}
            <div className="flex items-center gap-2 bg-slate-100 border border-slate-200 rounded-xl px-3 py-2.5 mb-2">
              <div className="w-8 h-8 bg-slate-500 rounded-full flex items-center justify-center text-white text-xs font-bold flex-shrink-0">帰</div>
              <div className="flex-1 min-w-0">
                <div className="text-xs font-semibold text-slate-700">到着（{rec.returnBase?.districtName}）</div>
                {displayReturn > 0 && <div className="text-xs text-blue-600 font-semibold">{rec.stops.length>0 ? rec.stops[rec.stops.length-1].districtName : rec.base?.districtName} から {displayReturn} km{routeData.legDurations[rec.stops.length] > 0 ? `（約${routeData.legDurations[rec.stops.length]}分）` : ''}</div>}
              </div>
              <div className="flex flex-shrink-0 gap-0.5">
                <button onClick={()=>setFavMemoModal(rec.returnBase)} title="お気に入り登録"
                  className="w-8 h-8 flex items-center justify-center text-slate-300 hover:text-yellow-400 active:text-yellow-500 text-lg">☆</button>
                <button onClick={()=>setMapMode("end")} title="地図から選択"
                  className={`w-8 h-8 flex items-center justify-center rounded-lg text-sm ${mapMode==="end"?"bg-slate-300 text-slate-800":"text-slate-500 hover:bg-slate-200"}`}>📍</button>
                <button onClick={()=>setShowBaseEdit("end")} title="検索して変更"
                  className="w-8 h-8 flex items-center justify-center text-slate-400 hover:text-slate-600 text-sm">🔍</button>
              </div>
            </div>
```

変更後（タイムラインラッパーの閉じタグ `</div>` も追加する）:
```jsx
            {/* 帰社 */}
            <div className="tl-node">
              <div className="tl-dot" style={{background:'#64748b'}}>帰</div>
              <div className="flex items-center gap-2 bg-slate-50 border border-slate-200 rounded-xl px-3 py-2.5 mb-2">
                <div className="flex-1 min-w-0">
                  <div className="text-xs font-semibold text-slate-600">到着（{rec.returnBase?.districtName}）</div>
                  {displayReturn > 0 && <div className="text-xs text-blue-600 font-semibold">{rec.stops.length>0 ? rec.stops[rec.stops.length-1].districtName : rec.base?.districtName} から {displayReturn} km{routeData.legDurations[rec.stops.length] > 0 ? `（約${routeData.legDurations[rec.stops.length]}分）` : ''}</div>}
                </div>
                <div className="flex flex-shrink-0 gap-1.5">
                  <button onClick={()=>setFavMemoModal(rec.returnBase)} className="act-btn act-btn-fav">★ 登録</button>
                  <button onClick={()=>setMapMode("end")} className={`act-btn ${mapMode==="end"?"act-btn-map":"act-btn-gray"}`}>📍 地図</button>
                </div>
              </div>
            </div>

            </div>{/* /tl-wrap */}
```

- [ ] **Step 3: ブラウザでリロードし、縦線と丸バッジが出発・帰社カードの左に表示されることを確認する**

- [ ] **Step 4: コミット**

```bash
git add index.html
git commit -m "feat: wrap stop list in timeline, update base/return cards"
```

---

## Task 4: 挿入ボタンを大型の展開式に変更する（3箇所）

**Files:**
- Modify: `index.html` — 挿入ボタン箇所（出発直後・各ストップ後・リスト末尾）

挿入ボタンは現在3箇所にある。すべて同じパターンに置き換える：
- 「＋ ここに訪問先を追加」大型ボタン（`tl-insert-btn`）をクリックすると `insertOpenAt` をセット
- `insertOpenAt` が一致するとき、下に2択（お気に入り/地図）を展開表示

- [ ] **Step 1: 出発直後の挿入ゾーン（`insert-base`）を置き換える**

変更前（約2147〜2173行）:
```jsx
            {/* 出発地の直後に挿入ボタン（1件目の前） */}
            {rec.stops.length < 10 && (
              <div className="mb-1.5 ml-10 pl-1">
                <div className="flex gap-1.5">
                  <button
                    onClick={()=>{ setInsertAfterIdx(-1); setShowFavs(true); }}
                    className="text-[11px] text-blue-500 border border-blue-200 rounded-full px-2.5 py-0.5 active:bg-blue-50">
                    お気に入りから追加
                  </button>
                  <button
                    onClick={()=>{ setInsertAfterIdx(-1); setMapMode('stop'); setShowMapPopup('insert-base'); }}
                    className="text-[11px] text-emerald-600 border border-emerald-200 rounded-full px-2.5 py-0.5 active:bg-emerald-50">
                    地図から追加
                  </button>
                </div>
                {showMapPopup === 'insert-base' && (
                  <div className="mt-1.5 bg-emerald-50 border border-emerald-300 rounded-xl px-3 py-2.5 flex items-start gap-2">
                    <span className="text-lg flex-shrink-0">📍</span>
                    <div>
                      <div className="text-xs font-bold text-emerald-700">地図を長押しして追加</div>
                      <div className="text-[11px] text-emerald-600 mt-0.5">地図タブに切り替えて、追加したい場所を長押ししてください。</div>
                    </div>
                    <button onClick={()=>setShowMapPopup(false)} className="ml-auto text-emerald-400 text-lg leading-none flex-shrink-0">×</button>
                  </div>
                )}
              </div>
            )}
```

変更後:
```jsx
            {/* 出発直後の挿入ゾーン */}
            {rec.stops.length < 10 && (
              <div className="mb-1">
                {insertOpenAt !== 'base' ? (
                  <button className="tl-insert-btn" onClick={()=>setInsertOpenAt('base')}>
                    <span style={{fontSize:'16px',lineHeight:1}}>＋</span> ここに訪問先を追加
                  </button>
                ) : (
                  <div className="tl-insert-choices">
                    <button
                      onClick={()=>{ setInsertAfterIdx(-1); setShowFavs(true); setInsertOpenAt(null); }}
                      style={{borderColor:'#93c5fd',color:'#2563eb',background:'#eff6ff'}}>
                      ☆ お気に入りから
                    </button>
                    <button
                      onClick={()=>{ setInsertAfterIdx(-1); setMapMode('stop'); setShowMapPopup('insert-base'); setInsertOpenAt(null); }}
                      style={{borderColor:'#6ee7b7',color:'#059669',background:'#f0fdf4'}}>
                      📍 地図から
                    </button>
                    <button onClick={()=>setInsertOpenAt(null)}
                      style={{flex:'0 0 auto',borderColor:'#e2e8f0',color:'#94a3b8',background:'#f8fafc',padding:'7px 10px'}}>
                      ✕
                    </button>
                  </div>
                )}
                {showMapPopup === 'insert-base' && (
                  <div className="mt-1.5 bg-emerald-50 border border-emerald-300 rounded-xl px-3 py-2.5 flex items-start gap-2">
                    <span className="text-lg flex-shrink-0">📍</span>
                    <div>
                      <div className="text-xs font-bold text-emerald-700">地図を長押しして追加</div>
                      <div className="text-[11px] text-emerald-600 mt-0.5">地図タブに切り替えて、追加したい場所を長押ししてください。</div>
                    </div>
                    <button onClick={()=>setShowMapPopup(false)} className="ml-auto text-emerald-400 text-lg leading-none flex-shrink-0">×</button>
                  </div>
                )}
              </div>
            )}
```

- [ ] **Step 2: 各ストップ内の挿入ゾーン（ループ内）を置き換える**

変更前（約2217〜2242行、`rec.stops.map` ループの内部）:
```jsx
                {rec.stops.length < 10 && (
                  <div className="mt-1.5 ml-10">
                    <div className="flex gap-1.5">
                      <button
                        onClick={()=>{ setInsertAfterIdx(i); setShowFavs(true); }}
                        className="text-[11px] text-blue-500 border border-blue-200 rounded-full px-2.5 py-0.5 active:bg-blue-50">
                        お気に入りから追加
                      </button>
                      <button
                        onClick={()=>{ setInsertAfterIdx(i); setMapMode('stop'); setShowMapPopup(`insert-${i}`); }}
                        className="text-[11px] text-emerald-600 border border-emerald-200 rounded-full px-2.5 py-0.5 active:bg-emerald-50">
                        地図から追加
                      </button>
                    </div>
                    {showMapPopup === `insert-${i}` && (
                      <div className="mt-1.5 bg-emerald-50 border border-emerald-300 rounded-xl px-3 py-2.5 flex items-start gap-2">
                        <span className="text-lg flex-shrink-0">📍</span>
                        <div>
                          <div className="text-xs font-bold text-emerald-700">地図を長押しして追加</div>
                          <div className="text-[11px] text-emerald-600 mt-0.5">地図タブに切り替えて、追加したい場所を長押ししてください。</div>
                        </div>
                        <button onClick={()=>setShowMapPopup(false)} className="ml-auto text-emerald-400 text-lg leading-none flex-shrink-0">×</button>
                      </div>
                    )}
                  </div>
                )}
```

変更後:
```jsx
                {rec.stops.length < 10 && (
                  <div className="mt-1">
                    {insertOpenAt !== i ? (
                      <button className="tl-insert-btn" onClick={()=>setInsertOpenAt(i)}>
                        <span style={{fontSize:'16px',lineHeight:1}}>＋</span> ここに訪問先を追加
                      </button>
                    ) : (
                      <div className="tl-insert-choices">
                        <button
                          onClick={()=>{ setInsertAfterIdx(i); setShowFavs(true); setInsertOpenAt(null); }}
                          style={{borderColor:'#93c5fd',color:'#2563eb',background:'#eff6ff'}}>
                          ☆ お気に入りから
                        </button>
                        <button
                          onClick={()=>{ setInsertAfterIdx(i); setMapMode('stop'); setShowMapPopup(`insert-${i}`); setInsertOpenAt(null); }}
                          style={{borderColor:'#6ee7b7',color:'#059669',background:'#f0fdf4'}}>
                          📍 地図から
                        </button>
                        <button onClick={()=>setInsertOpenAt(null)}
                          style={{flex:'0 0 auto',borderColor:'#e2e8f0',color:'#94a3b8',background:'#f8fafc',padding:'7px 10px'}}>
                          ✕
                        </button>
                      </div>
                    )}
                    {showMapPopup === `insert-${i}` && (
                      <div className="mt-1.5 bg-emerald-50 border border-emerald-300 rounded-xl px-3 py-2.5 flex items-start gap-2">
                        <span className="text-lg flex-shrink-0">📍</span>
                        <div>
                          <div className="text-xs font-bold text-emerald-700">地図を長押しして追加</div>
                          <div className="text-[11px] text-emerald-600 mt-0.5">地図タブに切り替えて、追加したい場所を長押ししてください。</div>
                        </div>
                        <button onClick={()=>setShowMapPopup(false)} className="ml-auto text-emerald-400 text-lg leading-none flex-shrink-0">×</button>
                      </div>
                    )}
                  </div>
                )}
```

- [ ] **Step 3: リスト末尾の追加ボタン（お気に入り/地図）を置き換える**

変更前（約2246〜2270行）:
```jsx
            {/* お気に入りから追加 */}
            {rec.stops.length < 10 && (
              <div className="mb-2">
                <div className="flex gap-2">
                  <button onClick={()=>setShowFavs(true)}
                    className="flex-1 border-2 border-dashed border-slate-200 rounded-xl py-2.5 text-sm text-slate-400 active:bg-blue-50 active:border-blue-400 active:text-blue-500">
                    ☆ お気に入りから追加
                  </button>
                  <button onClick={()=>{ setMapMode('stop'); setShowMapPopup('add'); }}
                    className="flex-1 border-2 border-dashed border-emerald-200 rounded-xl py-2.5 text-sm text-emerald-400 active:bg-emerald-50 active:border-emerald-400 active:text-emerald-600">
                    📍 地図から追加
                  </button>
                </div>
                {showMapPopup === 'add' && (
                  <div className="mt-2 bg-emerald-50 border border-emerald-300 rounded-xl px-3 py-2.5 flex items-start gap-2">
                    <span className="text-lg flex-shrink-0">📍</span>
                    <div>
                      <div className="text-xs font-bold text-emerald-700">地図を長押しして追加</div>
                      <div className="text-[11px] text-emerald-600 mt-0.5">地図タブに切り替えて、追加したい場所を長押ししてください。</div>
                    </div>
                    <button onClick={()=>setShowMapPopup(false)} className="ml-auto text-emerald-400 text-lg leading-none flex-shrink-0">×</button>
                  </div>
                )}
              </div>
            )}
```

変更後（帰社カードの `tl-node` の直前に配置する）:
```jsx
            {/* リスト末尾の追加ゾーン */}
            {rec.stops.length < 10 && (
              <div className="mb-1">
                {insertOpenAt !== 'end' ? (
                  <button className="tl-insert-btn" onClick={()=>setInsertOpenAt('end')}>
                    <span style={{fontSize:'16px',lineHeight:1}}>＋</span> ここに訪問先を追加
                  </button>
                ) : (
                  <div className="tl-insert-choices">
                    <button
                      onClick={()=>{ setInsertAfterIdx(null); setShowFavs(true); setInsertOpenAt(null); }}
                      style={{borderColor:'#93c5fd',color:'#2563eb',background:'#eff6ff'}}>
                      ☆ お気に入りから
                    </button>
                    <button
                      onClick={()=>{ setInsertAfterIdx(null); setMapMode('stop'); setShowMapPopup('add'); setInsertOpenAt(null); }}
                      style={{borderColor:'#6ee7b7',color:'#059669',background:'#f0fdf4'}}>
                      📍 地図から
                    </button>
                    <button onClick={()=>setInsertOpenAt(null)}
                      style={{flex:'0 0 auto',borderColor:'#e2e8f0',color:'#94a3b8',background:'#f8fafc',padding:'7px 10px'}}>
                      ✕
                    </button>
                  </div>
                )}
                {showMapPopup === 'add' && (
                  <div className="mt-1.5 bg-emerald-50 border border-emerald-300 rounded-xl px-3 py-2.5 flex items-start gap-2">
                    <span className="text-lg flex-shrink-0">📍</span>
                    <div>
                      <div className="text-xs font-bold text-emerald-700">地図を長押しして追加</div>
                      <div className="text-[11px] text-emerald-600 mt-0.5">地図タブに切り替えて、追加したい場所を長押ししてください。</div>
                    </div>
                    <button onClick={()=>setShowMapPopup(false)} className="ml-auto text-emerald-400 text-lg leading-none flex-shrink-0">×</button>
                  </div>
                )}
              </div>
            )}
```

- [ ] **Step 4: ブラウザでリロードし動作確認する**

確認項目：
- 「＋ ここに訪問先を追加」ボタンが各カード間に表示される
- クリックすると「☆ お気に入りから」「📍 地図から」「✕」の3択が展開される
- ✕ で閉じる
- お気に入りモーダルが開くこと
- 地図指示ポップアップが出ること

- [ ] **Step 5: コミット**

```bash
git add index.html
git commit -m "feat: replace small insert pills with large expandable insert buttons"
```

---

## Task 5: 訪問先カードのドラッグハンドルと☆/×ボタンを改善する

**Files:**
- Modify: `index.html:2186-2211` — `rec.stops.map` ループ内のカードJSX

- [ ] **Step 1: ドラッグハンドルを枠付きボックスに置き換える**

変更前（約2188〜2197行）:
```jsx
                  {/* ドラッグハンドル（PC＆スマホ共通） */}
                  <div
                    className="w-6 flex-shrink-0 flex flex-col items-center justify-center gap-0.5 pt-1.5 cursor-grab active:cursor-grabbing touch-none select-none"
                    onTouchStart={e=>onHandleTouchStart(e,i)}
                    onTouchMove={onHandleTouchMove}
                    onTouchEnd={onHandleTouchEnd}
                  >
                    <span className="block w-4 h-0.5 bg-slate-300 rounded"></span>
                    <span className="block w-4 h-0.5 bg-slate-300 rounded"></span>
                    <span className="block w-4 h-0.5 bg-slate-300 rounded"></span>
                  </div>
```

変更後:
```jsx
                  {/* ドラッグハンドル（枠付き） */}
                  <div
                    className="drag-box"
                    onTouchStart={e=>onHandleTouchStart(e,i)}
                    onTouchMove={onHandleTouchMove}
                    onTouchEnd={onHandleTouchEnd}
                  >
                    <span></span>
                    <span></span>
                    <span></span>
                  </div>
```

- [ ] **Step 2: ☆/× ボタンをラベル付き `act-btn` に置き換える**

変更前（約2205〜2210行）:
```jsx
                  <div className="flex flex-shrink-0">
                    <button onClick={()=>setFavMemoModal(s)} title="お気に入り登録"
                      className="w-9 h-9 flex items-center justify-center text-slate-300 hover:text-yellow-400 active:text-yellow-500 text-lg">☆</button>
                    <button onClick={()=>removeStop(i)} title="削除"
                      className="w-9 h-9 flex items-center justify-center text-slate-300 active:text-red-500 text-xl">×</button>
                  </div>
```

変更後:
```jsx
                  <div className="flex flex-shrink-0 gap-1.5">
                    <button onClick={()=>setFavMemoModal(s)} className="act-btn act-btn-fav">★ お気に入り</button>
                    <button onClick={()=>removeStop(i)} className="act-btn act-btn-del">✕ 削除</button>
                  </div>
```

- [ ] **Step 3: 訪問先カードに `tl-node` と `tl-dot` を追加する**

変更前（約2177〜2184行、`stop-item` の外側 `div`）:
```jsx
              <div key={`${s.lat}${s.lng}${i}`}
                ref={el => stopItemRefs.current[i] = el}
                className={`stop-item border rounded-xl px-3 py-2.5 mb-2 transition-all ${
                  draggingIdx === i ? 'opacity-40 scale-95 bg-blue-50 border-blue-300' :
                  dragOverIdxState === i && draggingIdx !== null ? 'border-blue-400 border-2 bg-blue-50' :
                  'bg-slate-50 border-slate-200'
                }`}
                draggable onDragStart={()=>onDragStart(i)} onDragOver={e=>onDragOver(e,i)} onDrop={e=>onDrop(e,i)} onDragEnd={onDragEnd}
              >
```

変更後（`tl-node` で囲み、`tl-dot` を追加）:
```jsx
              <div key={`${s.lat}${s.lng}${i}`} className="tl-node">
                <div className="tl-dot" style={{background:['#3b82f6','#f97316','#8b5cf6','#10b981','#ef4444','#0ea5e9','#ec4899','#eab308'][i % 8]}}>
                  {i+1}
                </div>
                <div
                  ref={el => stopItemRefs.current[i] = el}
                  className={`stop-item border rounded-xl px-3 py-2.5 mb-1 transition-all ${
                    draggingIdx === i ? 'opacity-40 scale-95 bg-blue-50 border-blue-300' :
                    dragOverIdxState === i && draggingIdx !== null ? 'border-blue-400 border-2 bg-blue-50' :
                    'bg-white border-slate-200'
                  }`}
                  draggable onDragStart={()=>onDragStart(i)} onDragOver={e=>onDragOver(e,i)} onDrop={e=>onDrop(e,i)} onDragEnd={onDragEnd}
                >
```

また、この `<div>` の閉じタグ（現在 `</div>` が2つ: stop-item の閉じ と map の閉じ）を 3つに増やす（`tl-node` 分）：

変更前（`</div>` が2つ — stop-item閉じ と stops.map閉じ）:
```jsx
              </div>
            ))}
```

変更後（`tl-node` の閉じも追加）:
```jsx
                </div>{/* /stop-item */}
              </div>{/* /tl-node */}
            ))}
```

- [ ] **Step 4: ブラウザでリロードし動作確認する**

確認項目：
- 各訪問先カードの左に色付き番号バッジが表示される
- ドラッグハンドルが枠付きボックスで表示される
- 「★ お気に入り」「✕ 削除」ボタンにラベルが付いている
- ドラッグによる並び替えが引き続き動作する（PC: drag&drop, スマホ: タッチドラッグ）
- お気に入り登録モーダルが開く
- 削除ボタンで訪問先が削除される

- [ ] **Step 5: コミット**

```bash
git add index.html
git commit -m "feat: improve drag handle, add labels to fav/delete buttons, add tl-node dots to stops"
```

---

## 最終確認チェックリスト

- [ ] 縦のタイムライン線が出発から帰社まで繋がっている
- [ ] 発・各番号・帰の色付き丸バッジが正しい位置に表示される
- [ ] 挿入ボタンが各カード間に「＋ ここに訪問先を追加」として表示される
- [ ] 挿入ボタンをクリックすると2択が展開され、✕で閉じられる
- [ ] ☆・× ボタンが「★ お気に入り」「✕ 削除」ラベル付きで表示される
- [ ] ドラッグハンドルが枠付きボックスで表示される
- [ ] モバイル（画面幅768px以下）でもレイアウトが崩れない
- [ ] 既存のドラッグ並び替えが動作する
- [ ] お気に入り追加・地図ピン追加が動作する
