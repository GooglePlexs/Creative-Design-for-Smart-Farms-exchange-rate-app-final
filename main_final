import json
import os
import time
import urllib.request

from fastapi import FastAPI, HTTPException, Query
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import HTMLResponse

app = FastAPI(title="Global Exchange Rate Calculator")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

# ---------------------------------------------------------------
# 환율 데이터
# open.er-api.com / API 키 불필요
# 서버에서 1시간 동안 캐시
# ---------------------------------------------------------------
API_URL = "https://open.er-api.com/v6/latest/USD"
CACHE_SECONDS = 3600

# API 연결 실패 시 사용하는 예비 환율
FALLBACK_RATES = {
    "USD": 1.0,
    "KRW": 1380.0,
    "EUR": 0.92,
    "JPY": 150.0,
    "CNY": 7.2,
    "GBP": 0.78,
    "AUD": 1.5,
    "CAD": 1.36,
    "CHF": 0.88,
    "HKD": 7.8,
}

_cache = {
    "rates": None,
    "updated": None,
    "fetched_at": 0.0,
    "live": False,
}


def get_rates() -> dict:
    now = time.time()

    if (
        _cache["rates"]
        and now - _cache["fetched_at"] < CACHE_SECONDS
    ):
        return _cache

    try:
        with urllib.request.urlopen(
            API_URL,
            timeout=10
        ) as res:
            data = json.loads(
                res.read().decode()
            )

        if data.get("result") != "success":
            raise ValueError("API error")

        _cache.update(
            rates=data["rates"],
            updated=data.get(
                "time_last_update_utc"
            ),
            fetched_at=now,
            live=True,
        )

    except Exception:

        # 실패 시 기존 캐시 사용
        # 캐시도 없으면 예비 환율 사용
        if not _cache["rates"]:

            _cache.update(
                rates=FALLBACK_RATES,
                updated=None,
                fetched_at=(
                    now
                    - CACHE_SECONDS
                    + 60
                ),
                live=False,
            )

    return _cache


# ---------------------------------------------------------------
# 지원 통화 목록 API
# ---------------------------------------------------------------

@app.get("/api/currencies")
def currencies():

    c = get_rates()

    return {
        "currencies":
            sorted(
                c["rates"].keys()
            ),

        "updated":
            c["updated"],

        "live":
            c["live"],
    }


# ---------------------------------------------------------------
# 환율 계산 API
# ---------------------------------------------------------------

@app.get("/api/convert")
def convert(
    amount: float,
    to: str,
    frm: str = Query(
        ...,
        alias="from"
    ),
):

    return _convert(
        amount,
        frm,
        to,
    )


def _convert(
    amount: float,
    frm: str,
    to: str,
):

    c = get_rates()

    rates = c["rates"]

    frm = frm.upper()
    to = to.upper()

    if (
        frm not in rates
        or to not in rates
    ):

        raise HTTPException(
            status_code=400,
            detail="Unsupported currency",
        )

    # USD 기준 환율을 이용해
    # 다른 통화끼리 환율 계산
    rate = (
        rates[to]
        / rates[frm]
    )

    return {

        "amount":
            amount,

        "from":
            frm,

        "to":
            to,

        "rate":
            rate,

        "result":
            amount * rate,

        "updated":
            c["updated"],

        "live":
            c["live"],
    }


# ---------------------------------------------------------------
# 홈페이지 HTML
# ---------------------------------------------------------------

HTML = """
<!DOCTYPE html>

<html lang="ko">

<head>

  <meta charset="UTF-8">

  <meta
    name="viewport"
    content="width=device-width, initial-scale=1.0"
  >

  <title>
    환율 계산기
  </title>


  <style>

    * {
      box-sizing: border-box;
    }


    body {

      margin: 0;

      min-height: 100vh;

      font-family:
        Arial,
        "Noto Sans KR",
        sans-serif;

      background:
        linear-gradient(
          135deg,
          #e0e7ff,
          #f8fafc
        );

      color: #0f172a;

      display: flex;

      align-items: center;

      justify-content: center;

      padding: 24px;

    }


    .container {

      width: 100%;

      max-width: 500px;

    }


    .top-bar {

      display: flex;

      justify-content: flex-end;

      margin-bottom: 12px;

    }


    #lang {

      width: auto;

      min-width: 110px;

      padding: 9px 12px;

      border:
        1px solid
        #cbd5e1;

      border-radius: 10px;

      background:
        rgba(
          255,
          255,
          255,
          0.9
        );

      color: #334155;

      font-size: 14px;

    }


    .card {

      background:
        rgba(
          255,
          255,
          255,
          0.96
        );

      border-radius: 24px;

      padding: 38px;

      box-shadow:
        0
        20px
        55px
        rgba(
          15,
          23,
          42,
          0.13
        );

      border:
        1px solid
        rgba(
          226,
          232,
          240,
          0.8
        );

    }


    .logo {

      text-align: center;

      font-size: 56px;

      margin-bottom: 4px;

    }


    h1 {

      margin: 0;

      text-align: center;

      font-size: 30px;

      letter-spacing: -0.8px;

    }


    .description {

      text-align: center;

      color: #64748b;

      margin-top: 10px;

      margin-bottom: 30px;

      font-size: 14px;

    }


    label {

      display: block;

      margin-top: 18px;

      margin-bottom: 8px;

      color: #334155;

      font-size: 14px;

      font-weight: 700;

    }


    input,
    select {

      width: 100%;

      padding: 13px 14px;

      border:
        1px solid
        #cbd5e1;

      border-radius: 11px;

      background: #ffffff;

      color: #0f172a;

      font-size: 15px;

      transition:
        border-color
        0.15s ease,
        box-shadow
        0.15s ease;

    }


    input:focus,
    select:focus {

      outline: none;

      border-color:
        #2563eb;

      box-shadow:
        0 0 0 3px
        rgba(
          37,
          99,
          235,
          0.10
        );

    }


    .currency-search {

      margin-bottom: 7px;

      background:
        #f8fafc;

    }


    .search-hint {

      margin:
        0
        0
        7px
        2px;

      color: #94a3b8;

      font-size: 12px;

    }


    .currency-box {

      background:
        #f8fafc;

      border:
        1px solid
        #e2e8f0;

      border-radius: 14px;

      padding: 14px;

      margin-top: 4px;

    }


    .swap {

      width: 46px;

      height: 46px;

      display: block;

      margin:
        16px auto 2px;

      border: none;

      border-radius: 50%;

      background:
        #0f172a;

      color: #ffffff;

      font-size: 22px;

      cursor: pointer;

      box-shadow:
        0
        8px
        18px
        rgba(
          15,
          23,
          42,
          0.14
        );

      transition:
        transform
        0.15s ease,
        background
        0.15s ease;

    }


    .swap:hover {

      background:
        #1e293b;

      transform:
        translateY(-1px);

    }


    .result-card {

      margin-top: 24px;

      padding: 22px;

      border-radius: 16px;

      background:
        #eff6ff;

      border:
        1px solid
        #dbeafe;

      text-align: center;

    }


    .result {

      font-size: 27px;

      font-weight: 800;

      color: #2563eb;

      letter-spacing:
        -0.5px;

    }


    .sub {

      font-size: 13px;

      color: #64748b;

      margin-top: 8px;

      line-height: 1.5;

    }


    .api-link {

      margin-top: 22px;

      text-align: center;

    }


    .api-link a {

      color: #2563eb;

      text-decoration: none;

      font-size: 13px;

      font-weight: 700;

    }


    footer {

      margin-top: 18px;

      text-align: center;

      font-size: 11px;

      color: #94a3b8;

    }


    footer a {

      color: #64748b;

      text-decoration: none;

    }


    @media (
      max-width: 520px
    ) {

      body {

        padding: 14px;

        align-items:
          flex-start;

      }


      .container {

        margin-top: 16px;

      }


      .card {

        padding:
          26px
          20px;

        border-radius:
          20px;

      }


      h1 {

        font-size:
          26px;

      }


      .result {

        font-size:
          23px;

      }

    }

  </style>

</head>


<body>


  <div class="container">


    <div class="top-bar">


      <select
        id="lang"
        onchange="setLang(this.value)"
      >

        <option value="ko">
          한국어
        </option>

        <option value="en">
          English
        </option>

        <option value="ja">
          日本語
        </option>

        <option value="zh">
          中文
        </option>

        <option value="es">
          Español
        </option>

      </select>


    </div>


    <div class="card">


      <div class="logo">
        💱
      </div>


      <h1 id="title">
      </h1>


      <div class="description">

        Global FastAPI
        Exchange Rate Calculator

      </div>


      <!-- 금액 -->


      <label id="lAmount">
      </label>


      <input
        id="amount"
        type="number"
        step="any"
        value="100"
        oninput="convert()"
      >


      <!-- 보내는 통화 -->


      <label id="lFrom">
      </label>


      <div class="currency-box">


        <input
          id="fromSearch"
          class="currency-search"
          type="search"
          autocomplete="off"
          placeholder="KRW, USD..."
          oninput="filterCurrency('from')"
        >


        <div
          id="fromSearchHint"
          class="search-hint"
        >
        </div>


        <select
          id="from"
          onchange="convert()"
        >
        </select>


      </div>


      <!-- 통화 스왑 -->


      <button
        class="swap"
        onclick="swap()"
        aria-label="Swap currencies"
        title="Swap currencies"
      >

        ⇅

      </button>


      <!-- 받는 통화 -->


      <label id="lTo">
      </label>


      <div class="currency-box">


        <input
          id="toSearch"
          class="currency-search"
          type="search"
          autocomplete="off"
          placeholder="KRW, USD..."
          oninput="filterCurrency('to')"
        >


        <div
          id="toSearchHint"
          class="search-hint"
        >
        </div>


        <select
          id="to"
          onchange="convert()"
        >
        </select>


      </div>


      <!-- 계산 결과 -->


      <div class="result-card">


        <div
          id="result"
          class="result"
        >
        </div>


        <div
          id="rate"
          class="sub"
        >
        </div>


        <div
          id="updated"
          class="sub"
        >
        </div>


      </div>


      <div class="api-link">


        <a
          href="/docs"
          target="_blank"
        >

          FastAPI API 문서 보기

        </a>


      </div>


      <footer>


        <a
          href="https://www.exchangerate-api.com"
          target="_blank"
          rel="noopener"
        >

          Rates By Exchange Rate API

        </a>


      </footer>


    </div>


  </div>


  <script>


    // ---------------------------------------------------------
    // 다국어 텍스트
    // ---------------------------------------------------------


    const T = {


      ko: {

        title:
          "환율 계산기",

        amount:
          "금액",

        from:
          "보내는 통화",

        to:
          "받는 통화",

        rate:
          "환율",

        updated:
          "기준 시각",

        search:
          "통화 검색 (예: KRW, USD)",

        searchHint:
          "영문 통화 코드 또는 통화명으로 검색할 수 있습니다.",

        noResult:
          "검색 결과 없음",

        fallback:
          "※ 실시간 환율을 불러오지 못해 예비 환율을 사용 중입니다.",

        err:
          "계산할 수 없습니다."

      },


      en: {

        title:
          "Exchange Rate Calculator",

        amount:
          "Amount",

        from:
          "From",

        to:
          "To",

        rate:
          "Rate",

        updated:
          "Updated",

        search:
          "Search currency (e.g. KRW, USD)",

        searchHint:
          "Search by currency code or currency name.",

        noResult:
          "No matching currency",

        fallback:
          "* Live rates unavailable. Showing approximate fallback rates.",

        err:
          "Cannot convert."

      },


      ja: {

        title:
          "為替レート計算機",

        amount:
          "金額",

        from:
          "換算元",

        to:
          "換算先",

        rate:
          "レート",

        updated:
          "更新日時",

        search:
          "通貨を検索 (例: KRW, USD)",

        searchHint:
          "通貨コードまたは通貨名で検索できます。",

        noResult:
          "該当する通貨がありません",

        fallback:
          "※ リアルタイムのレートを取得できないため、概算レートを表示しています。",

        err:
          "計算できません。"

      },


      zh: {

        title:
          "汇率计算器",

        amount:
          "金额",

        from:
          "源货币",

        to:
          "目标货币",

        rate:
          "汇率",

        updated:
          "更新时间",

        search:
          "搜索货币（例如 KRW、USD）",

        searchHint:
          "可按货币代码或货币名称搜索。",

        noResult:
          "没有匹配的货币",

        fallback:
          "* 无法获取实时汇率，当前显示为近似汇率。",

        err:
          "无法计算。"

      },


      es: {

        title:
          "Calculadora de divisas",

        amount:
          "Cantidad",

        from:
          "De",

        to:
          "A",

        rate:
          "Tipo de cambio",

        updated:
          "Actualizado",

        search:
          "Buscar divisa (p. ej. KRW, USD)",

        searchHint:
          "Busca por código o nombre de divisa.",

        noResult:
          "No hay divisas coincidentes",

        fallback:
          "* Tipos en vivo no disponibles. Se muestran valores aproximados.",

        err:
          "No se puede convertir."

      },

    };


    const DEFAULT_TO = {

      ko:
        "KRW",

      en:
        "EUR",

      ja:
        "JPY",

      zh:
        "CNY",

      es:
        "EUR",

    };


    // 자주 사용하는 통화를
    // 전체 목록의 위쪽에 표시
    const POPULAR = [

      "USD",
      "KRW",
      "EUR",
      "JPY",
      "CNY",
      "GBP",
      "AUD",
      "CAD",
      "CHF",
      "HKD",

    ];


    // ---------------------------------------------------------
    // 사용자 브라우저 언어 확인
    // ---------------------------------------------------------


    let lang =
      (
        navigator.language
        || "en"
      ).slice(
        0,
        2
      );


    if (
      !T[lang]
    ) {

      lang =
        "en";

    }


    let codes =
      [];


    // ---------------------------------------------------------
    // 통화 이름 가져오기
    // ---------------------------------------------------------


    function name(
      code
    ) {

      try {

        return new Intl.DisplayNames(

          [lang],

          {
            type:
              "currency"
          }

        ).of(
          code
        );

      }

      catch {

        return code;

      }

    }


    // ---------------------------------------------------------
    // 통화 정렬
    // ---------------------------------------------------------


    function orderedCodes() {

      return [

        ...POPULAR.filter(
          c =>
            codes.includes(c)
        ),

        ...codes.filter(
          c =>
            !POPULAR.includes(c)
        ),

      ];

    }


    function currencyLabel(
      code
    ) {

      return (
        `${code} - ${name(code)}`
      );

    }


    // ---------------------------------------------------------
    // 검색 결과를 드롭다운에 넣기
    // ---------------------------------------------------------


    function fillOneSelect(
      selectId,
      searchId,
      keepValue
    ) {


      const select =
        document.getElementById(
          selectId
        );


      const search =
        document.getElementById(
          searchId
        );


      const q =
        (
          search.value
          || ""
        )
        .trim()
        .toLowerCase();


      /*
        검색 방식

        k
        → 코드에 k가 포함된 통화

        kr
        → 코드에 kr이 포함된 통화

        krw
        → KRW가 최우선

        usd
        → USD가 최우선

        jpy
        → JPY가 최우선
      */


      const filtered =
        orderedCodes()

        .filter(
          code => {

            const codeText =
              code.toLowerCase();


            const nameText =
              (
                name(code)
                || ""
              ).toLowerCase();


            return (

              !q

              ||

              codeText.includes(
                q
              )

              ||

              nameText.includes(
                q
              )

            );

          }
        )


        // 검색 결과 우선순위
        .sort(
          (
            a,
            b
          ) => {


            if (
              !q
            ) {

              return 0;

            }


            const aCode =
              a.toLowerCase();


            const bCode =
              b.toLowerCase();


            const aName =
              (
                name(a)
                || ""
              ).toLowerCase();


            const bName =
              (
                name(b)
                || ""
              ).toLowerCase();


            function rank(
              codeText,
              nameText
            ) {


              // 완전 일치
              if (
                codeText === q
              ) {

                return 0;

              }


              // 코드가 검색어로 시작
              if (
                codeText.startsWith(
                  q
                )
              ) {

                return 1;

              }


              // 코드 중간 포함
              if (
                codeText.includes(
                  q
                )
              ) {

                return 2;

              }


              // 통화명이 검색어로 시작
              if (
                nameText.startsWith(
                  q
                )
              ) {

                return 3;

              }


              // 통화명 중간 포함
              if (
                nameText.includes(
                  q
                )
              ) {

                return 4;

              }


              return 5;

            }


            const rankDiff =

              rank(
                aCode,
                aName
              )

              -

              rank(
                bCode,
                bName
              );


            if (
              rankDiff !== 0
            ) {

              return rankDiff;

            }


            return a.localeCompare(
              b
            );

          }
        );


      // 검색 결과 없음
      if (
        !filtered.length
      ) {


        select.innerHTML =

          `<option value="">
            ${T[lang].noResult}
          </option>`;


        select.disabled =
          true;


        return;

      }


      select.disabled =
        false;


      select.innerHTML =

        filtered

        .map(
          code =>

            `<option value="${code}">
              ${currencyLabel(code)}
            </option>`

        )

        .join(
          ""
        );


      // 기존 선택값 유지
      if (

        keepValue

        &&

        filtered.includes(
          keepValue
        )

      ) {


        select.value =
          keepValue;

      }


      else {


        // KRW 같은 완전 일치 코드가 있으면
        // 그 통화를 바로 선택
        const exact =
          filtered.find(
            code =>
              code.toLowerCase()
              === q
          );


        select.value =
          exact
          || filtered[0];

      }

    }


    // ---------------------------------------------------------
    // 보내는 통화 / 받는 통화 목록 채우기
    // ---------------------------------------------------------


    function fillSelects(
      keepFrom,
      keepTo
    ) {


      fillOneSelect(

        "from",

        "fromSearch",

        keepFrom
        || "USD"

      );


      fillOneSelect(

        "to",

        "toSearch",

        keepTo
        || DEFAULT_TO[lang]
        || "EUR"

      );

    }


    // ---------------------------------------------------------
    // 통화 검색
    // ---------------------------------------------------------


    function filterCurrency(
      side
    ) {


      const selectId =

        side === "from"

        ? "from"

        : "to";


      const searchId =

        side === "from"

        ? "fromSearch"

        : "toSearch";


      const select =

        document.getElementById(
          selectId
        );


      const previous =
        select.value;


      fillOneSelect(

        selectId,

        searchId,

        previous

      );


      if (
        select.value
      ) {


        convert();


      }


      else {


        document.getElementById(
          "result"
        ).textContent =
          "";


        document.getElementById(
          "rate"
        ).textContent =
          "";


        document.getElementById(
          "updated"
        ).textContent =
          "";


      }

    }


    // ---------------------------------------------------------
    // 언어 변경
    // ---------------------------------------------------------


    function setLang(
      l
    ) {


      lang =
        l;


      const t =
        T[l];


      document.documentElement.lang =
        l;


      document.getElementById(
        "lang"
      ).value =
        l;


      document.getElementById(
        "title"
      ).textContent =
        t.title;


      document.getElementById(
        "lAmount"
      ).textContent =
        t.amount;


      document.getElementById(
        "lFrom"
      ).textContent =
        t.from;


      document.getElementById(
        "lTo"
      ).textContent =
        t.to;


      document.getElementById(
        "fromSearch"
      ).placeholder =
        t.search;


      document.getElementById(
        "toSearch"
      ).placeholder =
        t.search;


      document.getElementById(
        "fromSearchHint"
      ).textContent =
        t.searchHint;


      document.getElementById(
        "toSearchHint"
      ).textContent =
        t.searchHint;


      if (
        codes.length
      ) {


        fillSelects(

          document.getElementById(
            "from"
          ).value,

          document.getElementById(
            "to"
          ).value

        );


        convert();


      }

    }


    // ---------------------------------------------------------
    // 숫자 출력 형식
    // ---------------------------------------------------------


    function fmt(
      n,
      max
    ) {


      return new Intl.NumberFormat(

        lang,

        {
          maximumFractionDigits:
            max
        }

      ).format(
        n
      );

    }


    // ---------------------------------------------------------
    // 환율 계산
    // ---------------------------------------------------------


    async function convert() {


      const amount =

        document.getElementById(
          "amount"
        ).value;


      const frm =

        document.getElementById(
          "from"
        ).value;


      const to =

        document.getElementById(
          "to"
        ).value;


      const out =

        document.getElementById(
          "result"
        );


      if (

        amount === ""

        ||

        !frm

        ||

        !to

      ) {


        out.textContent =
          "";


        return;

      }


      try {


        const res =

          await fetch(

            `/api/convert?amount=${encodeURIComponent(amount)}&from=${encodeURIComponent(frm)}&to=${encodeURIComponent(to)}`

          );


        if (
          !res.ok
        ) {


          out.textContent =
            T[lang].err;


          return;

        }


        const d =
          await res.json();


        out.textContent =

          `${fmt(d.amount, 4)} ${d.from} = ${fmt(d.result, 4)} ${d.to}`;


        document.getElementById(
          "rate"
        ).textContent =

          `${T[lang].rate}: 1 ${d.from} = ${fmt(d.rate, 6)} ${d.to}`;


        document.getElementById(
          "updated"
        ).textContent =

          d.live

          ?

          `${T[lang].updated}: ${d.updated || ""}`

          :

          T[lang].fallback;


      }


      catch {


        out.textContent =
          T[lang].err;


      }

    }


    // ---------------------------------------------------------
    // 보내는 통화 ↔ 받는 통화
    // ---------------------------------------------------------


    function swap() {


      const f =

        document.getElementById(
          "from"
        );


      const t =

        document.getElementById(
          "to"
        );


      const oldFrom =
        f.value;


      const oldTo =
        t.value;


      // 검색창 초기화
      document.getElementById(
        "fromSearch"
      ).value =
        "";


      document.getElementById(
        "toSearch"
      ).value =
        "";


      fillSelects(

        oldTo,

        oldFrom

      );


      convert();

    }


    // ---------------------------------------------------------
    // 사이트 처음 실행
    // ---------------------------------------------------------


    async function init() {


      setLang(
        lang
      );


      const res =

        await fetch(
          "/api/currencies"
        );


      const d =

        await res.json();


      codes =

        d.currencies;


      fillSelects();


      convert();

    }


    init();


  </script>


</body>

</html>
"""


# ---------------------------------------------------------------
# 메인 홈페이지
# ---------------------------------------------------------------

@app.get(
    "/",
    response_class=HTMLResponse
)
def home():

    return HTML


# ---------------------------------------------------------------
# Python으로 직접 실행할 경우
# ---------------------------------------------------------------

if __name__ == "__main__":

    import uvicorn

    uvicorn.run(
        app,
        host="0.0.0.0",
        port=int(
            os.environ.get(
                "PORT",
                8000
            )
        ),
    )
