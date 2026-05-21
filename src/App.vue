<script setup lang="ts">
import { ref, onMounted, onUnmounted, reactive, watch, computed } from "vue";

const DEBUG = false;

type Sheet = {
  telenorOpeningBalance: number | null;
  jazzOpeningBalance: number | null;
  ufoneOpeningBalance: number | null;
  zongOpeningBalance: number | null;

  telenorNewBalance: number | null;
  jazzNewBalance: number | null;
  ufoneNewBalance: number | null;
  zongNewBalance: number | null;

  telenorReversalBalance: number | null;
  jazzReversalBalance: number | null;
  ufoneReversalBalance: number | null;
  zongReversalBalance: number | null;

  telenorClosingBalance: number | null;
  jazzClosingBalance: number | null;
  ufoneClosingBalance: number | null;
  zongClosingBalance: number | null;

  accountBalance265999891: number | null;
  accountBalance266001445: number | null;
  accountBalance37300247: number | null;
  accountBalance257283991: number | null;

  borrow: { name: string; amount: number | null }[];
  recovery: { name: string; amount: number | null }[];

  deposit265999891: number | null;
  deposit266001445: number | null;
  deposit37300247: number | null;
  deposit257283991: number | null;

  withdrawl265999891: number | null;
  withdrawl266001445: number | null;
  withdrawl37300247: number | null;
  withdrawl257283991: number | null;

  totalCards: number | null;
  sellCards: number | null;
};

const sheet = reactive<Sheet>({
    telenorOpeningBalance: null,
    jazzOpeningBalance: null,
    ufoneOpeningBalance: null,
    zongOpeningBalance: null,

    telenorNewBalance: null,
    jazzNewBalance: null,
    ufoneNewBalance: null,
    zongNewBalance: null,

    telenorReversalBalance: null,
    jazzReversalBalance: null,
    ufoneReversalBalance: null,
    zongReversalBalance: null,

    telenorClosingBalance: null,
    jazzClosingBalance: null,
    ufoneClosingBalance: null,
    zongClosingBalance: null,

    accountBalance265999891: null,
    accountBalance266001445: null,
    accountBalance37300247: null,
    accountBalance257283991: null,

    borrow: Array.from({ length: 12 }, () => ({
        name: "",
        amount: null as number | null,
    })),

    recovery: Array.from({ length: 12 }, () => ({
        name: "",
        amount: null as number | null,
    })),

    deposit265999891: null,
    deposit266001445: null,
    deposit37300247: null,
    deposit257283991: null,

    withdrawl265999891: null,
    withdrawl266001445: null,
    withdrawl37300247: null,
    withdrawl257283991: null,

    totalCards: null,
    sellCards: null,
});

const savedSheet = localStorage.getItem("sheet");

if (savedSheet) {
    const parsed = JSON.parse(savedSheet);

    Object.assign(sheet, parsed);

    if (!parsed.borrow) {
        sheet.borrow = Array.from({ length: 12 }, () => ({
            name: "",
            amount: null,
        }));
    }

    if (!parsed.recovery) {
        sheet.recovery = Array.from({ length: 12 }, () => ({
            name: "",
            amount: null,
        }));
    }
}

watch(
    sheet,
    (newValue) => {
        localStorage.setItem(
            "sheet",
            JSON.stringify(newValue)
        );
    },
    {
        deep: true,
    }
);

const clearSheet = () => {
  Object.keys(sheet).forEach((key) => {
    const k = key as keyof Sheet;

    const value = sheet[k];

    if (Array.isArray(value)) {
      (sheet as any)[k] = value.map(() => ({
        name: "",
        amount: null,
      }));
    } else if (typeof value === "number" || value === null) {
      (sheet as any)[k] = null;
    }
  });

  localStorage.removeItem("sheet");
};

const n = (v: number | null | undefined) => v ?? 0;

// Header
const dateTime = ref({
    date: "",
    time: "",
});

const updateDateTime = () => {
    const now = new Date();

    dateTime.value.date = now.toLocaleDateString("en-US", {
        weekday: "long",
        year: "numeric",
        month: "long",
        day: "numeric",
    });

    dateTime.value.time = now.toLocaleTimeString("en-US", {
        hour: "2-digit",
        minute: "2-digit",
        second: "2-digit",
    });
};

let interval: number;

onMounted(() => {
    updateDateTime();

    interval = window.setInterval(() => {
        updateDateTime();
    }, 1000);

    const preventWheelOnNumberInputs = (e: WheelEvent) => {
        const target = e.target as HTMLElement;
        if (
            target instanceof HTMLInputElement &&
            target.type === "number"
        ) {
            target.blur();
        }
    };

    document.addEventListener("wheel", preventWheelOnNumberInputs, {
        passive: true,
    });
});

onUnmounted(() => {
    clearInterval(interval);
});

// Sheet
const getELoadSell = (type: "telenor" | "jazz" | "ufone" | "zong") => {
    return (
        (sheet[`${type}OpeningBalance`] || 0) +
        (sheet[`${type}NewBalance`] || 0) +
        (sheet[`${type}ReversalBalance`] || 0) -
        (sheet[`${type}ClosingBalance`] || 0)
    );
};

const totalELoad = computed(() => {
    const telenor =
        (sheet.telenorOpeningBalance || 0) +
        (sheet.telenorNewBalance || 0) +
        (sheet.telenorReversalBalance || 0) -
        (sheet.telenorClosingBalance || 0);

    const jazz =
        (sheet.jazzOpeningBalance || 0) +
        (sheet.jazzNewBalance || 0) +
        (sheet.jazzReversalBalance || 0) -
        (sheet.jazzClosingBalance || 0);

    const ufone =
        (sheet.ufoneOpeningBalance || 0) +
        (sheet.ufoneNewBalance || 0) +
        (sheet.ufoneReversalBalance || 0) -
        (sheet.ufoneClosingBalance || 0);

    const zong =
        (sheet.zongOpeningBalance || 0) +
        (sheet.zongNewBalance || 0) +
        (sheet.zongReversalBalance || 0) -
        (sheet.zongClosingBalance || 0);
    return telenor + jazz + ufone + zong;
});

const borrowTotal = computed(() => {
    return sheet.borrow.reduce(
        (sum, item) => sum + (Number(item.amount) || 0),
        0
    );
});

const recoveryTotal = computed(() => {
    return sheet.recovery.reduce(
        (sum, item) => sum + (Number(item.amount) || 0),
        0
    );
});

</script>

<template>
    <div id="print-area">
        <div class="container">
            <div v-if="DEBUG">
                <pre v-for="([key, value], index) in Object.entries(sheet)" :key="index">
                    <strong>{{ key }}:</strong>
                    {{ typeof value === 'object' ? JSON.stringify(value, null, 2) : value }}
                </pre>
            </div>

            <div class="header">
                <div class="title-container">
                    <h1>KMK Communication</h1>
                </div>
                <div class="time-actions-container">
                    <h2 id="current-date">{{ dateTime.date }}</h2>
                    <h2 id="current-time">{{ dateTime.time }}</h2>
                    <div class="actions no-print">
                        <button class="button" @click="console.log('Save PDF Clicked')">
                            Save PDF
                        </button>
                        <button class="button-danger" @click="clearSheet">
                            Clear Sheet
                        </button>
                    </div>
                </div>
            </div>

            <div class="eload-accounts-container">
                <div>
                    <div class="easyload-container">
                        <div class="easyload">
                            <table>
                                <tbody>
                                    <tr>
                                        <th />
                                        <th>Telenor</th>
                                        <th>Jazz</th>
                                        <th>Ufone</th>
                                        <th>Zong</th>
                                    </tr>

                                    <tr>
                                        <td>Opening Balance</td>
                                        <td><input v-model.number="sheet.telenorOpeningBalance" type="number" /></td>
                                        <td><input v-model.number="sheet.jazzOpeningBalance" type="number" /></td>
                                        <td><input v-model.number="sheet.ufoneOpeningBalance" type="number" /></td>
                                        <td><input v-model.number="sheet.zongOpeningBalance" type="number" /></td>
                                    </tr>

                                    <tr>
                                        <td>New Balance</td>
                                        <td><input v-model.number="sheet.telenorNewBalance" type="number" /></td>
                                        <td><input v-model.number="sheet.jazzNewBalance" type="number" /></td>
                                        <td><input v-model.number="sheet.ufoneNewBalance" type="number" /></td>
                                        <td><input v-model.number="sheet.zongNewBalance" type="number" /></td>
                                    </tr>

                                    <tr>
                                        <td>Reversal Balance</td>
                                        <td><input v-model.number="sheet.telenorReversalBalance" type="number" /></td>
                                        <td><input v-model.number="sheet.jazzReversalBalance" type="number" /></td>
                                        <td><input v-model.number="sheet.ufoneReversalBalance" type="number" /></td>
                                        <td><input v-model.number="sheet.zongReversalBalance" type="number" /></td>
                                    </tr>

                                    <tr>
                                        <td>Total Rs</td>
                                        <td><input
                                                :value="n(sheet.telenorOpeningBalance) + n(sheet.telenorNewBalance) + n(sheet.telenorReversalBalance)"
                                                type="number" disabled /></td>
                                        <td><input
                                                :value="n(sheet.jazzOpeningBalance) + n(sheet.jazzNewBalance) + n(sheet.jazzReversalBalance)"
                                                type="number" disabled /></td>
                                        <td><input
                                                :value="n(sheet.ufoneOpeningBalance) + n(sheet.ufoneNewBalance) + n(sheet.ufoneReversalBalance)"
                                                type="number" disabled /></td>
                                        <td><input
                                                :value="n(sheet.zongOpeningBalance) + n(sheet.zongNewBalance) + n(sheet.zongReversalBalance)"
                                                type="number" disabled /></td>
                                    </tr>

                                    <tr>
                                        <td>Closing Balance</td>
                                        <td><input v-model.number="sheet.telenorClosingBalance" type="number" /></td>
                                        <td><input v-model.number="sheet.jazzClosingBalance" type="number" /></td>
                                        <td><input v-model.number="sheet.ufoneClosingBalance" type="number" /></td>
                                        <td><input v-model.number="sheet.zongClosingBalance" type="number" /></td>
                                    </tr>

                                    <tr>
                                        <td>Total ELoad Sell</td>
                                        <td><input :value="getELoadSell('telenor')" type="number" disabled /></td>
                                        <td><input :value="getELoadSell('jazz')" type="number" disabled /></td>
                                        <td><input :value="getELoadSell('ufone')" type="number" disabled /></td>
                                        <td><input :value="getELoadSell('zong')" type="number" disabled /></td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>

                        <div class="eload-summery mx">
                            <table>
                                <tbody>
                                    <tr>
                                        <th colspan="2">Easyload Summery</th>
                                    </tr>
                                    <tr>
                                        <td>Telenor</td>
                                        <td><input :value="getELoadSell('telenor')" type="number" disabled /></td>
                                    </tr>
                                    <tr>
                                        <td>Jazz</td>
                                        <td><input :value="getELoadSell('jazz')" type="number" disabled /></td>
                                    </tr>
                                    <tr>
                                        <td>Ufone</td>
                                        <td><input :value="getELoadSell('ufone')" type="number" disabled /></td>
                                    </tr>
                                    <tr>
                                        <td>Zong</td>
                                        <td><input :value="getELoadSell('zong')" type="number" disabled /></td>
                                    </tr>
                                    <tr>
                                        <td>Total Load</td>
                                        <td><input :value="totalELoad" type="number" disabled /></td>
                                    </tr>
                                    <tr>
                                        <td>AD</td>
                                        <td><input :value="borrowTotal" type="number" disabled /></td>
                                    </tr>
                                    <tr>
                                        <td>Total</td>
                                        <td><input :value="totalELoad - borrowTotal" type="number" disabled /></td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>

                    <div class="accounts-container space-y">
                        <div class="accounts">
                            <table>
                                <tbody>
                                    <tr>
                                        <th>Accounts</th>
                                        <th>265999891</th>
                                        <th>266001445</th>
                                        <th>37300247</th>
                                        <th>257283991</th>
                                    </tr>

                                    <tr>
                                        <td>Rs</td>
                                        <td><input v-model="sheet.accountBalance265999891" type="number" /></td>
                                        <td><input v-model="sheet.accountBalance266001445" type="number" /></td>
                                        <td><input v-model="sheet.accountBalance37300247" type="number" /></td>
                                        <td><input v-model="sheet.accountBalance257283991" type="number" /></td>
                                    </tr>

                                    <tr>
                                        <td>Deposit</td>
                                        <td><input v-model="sheet.deposit265999891" type="number" /></td>
                                        <td><input v-model="sheet.deposit266001445" type="number" /></td>
                                        <td><input v-model="sheet.deposit37300247" type="number" /></td>
                                        <td><input v-model="sheet.deposit257283991" type="number" /></td>
                                    </tr>

                                    <tr>
                                        <td>Total Rs</td>
                                        <td><input :value="n(sheet.accountBalance265999891) + n(sheet.deposit265999891)"
                                                type="number" disabled /></td>
                                        <td><input :value="n(sheet.accountBalance266001445) + n(sheet.deposit266001445)"
                                                type="number" disabled /></td>
                                        <td><input :value="n(sheet.accountBalance37300247) + n(sheet.deposit37300247)"
                                                type="number" disabled /></td>
                                        <td><input :value="n(sheet.accountBalance257283991) + n(sheet.deposit257283991)"
                                                type="number" disabled /></td>
                                    </tr>

                                    <tr>
                                        <td>Withdrawl</td>
                                        <td><input v-model="sheet.withdrawl265999891" type="number" /></td>
                                        <td><input v-model="sheet.withdrawl266001445" type="number" /></td>
                                        <td><input v-model="sheet.withdrawl37300247" type="number" /></td>
                                        <td><input v-model="sheet.withdrawl257283991" type="number" /></td>
                                    </tr>

                                    <tr>
                                        <td>Remaining Balance</td>
                                        <td><input
                                                :value="(n(sheet.accountBalance265999891) + n(sheet.deposit265999891)) - n(sheet.withdrawl265999891)"
                                                type="number" disabled /></td>
                                        <td><input
                                                :value="(n(sheet.accountBalance266001445) + n(sheet.deposit266001445)) - n(sheet.withdrawl266001445)"
                                                type="number" disabled /></td>
                                        <td><input
                                                :value="(n(sheet.accountBalance37300247) + n(sheet.deposit37300247)) - n(sheet.withdrawl37300247)"
                                                type="number" disabled /></td>
                                        <td><input
                                                :value="(n(sheet.accountBalance257283991) + n(sheet.deposit257283991)) - n(sheet.withdrawl257283991)"
                                                type="number" disabled /></td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>

                        <div class="cards mx">
                            <table>
                                <tbody>
                                    <tr>
                                        <th colspan="2">Cards</th>
                                    </tr>
                                    <tr>
                                        <td>Total</td>
                                        <td><input v-model="sheet.totalCards" type="number" /></td>
                                    </tr>
                                    <tr>
                                        <td>Sell</td>
                                        <td><input v-model="sheet.sellCards" type="number" /></td>
                                    </tr>
                                    <tr>
                                        <td>Remaining</td>
                                        <td><input :value="n(sheet.totalCards) - n(sheet.sellCards)" type="number" disabled />
                                        </td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>
                </div>

                <div class="borrow-recovery-container">
                    <div class="borrow-container">
                        <table>
                            <tbody>
                                <tr>
                                    <th colspan="2">Borrow</th>
                                </tr>

                                <tr v-for="(row, index) in sheet.borrow" :key="index">
                                    <td>
                                        <input type="text" v-model="row.name" />
                                    </td>
                                    <td>
                                        <input type="number" v-model.number="row.amount" />
                                    </td>
                                </tr>
                                <tr>
                                    <td>Total</td>
                                    <td>
                                        <input type="number" :value="borrowTotal" disabled />
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>

                    <div class="recovery-container">
                        <table>
                            <tbody>
                                <tr>
                                    <th colspan="2">Recovery</th>
                                </tr>

                                <tr v-for="(row, index) in sheet.recovery" :key="index">
                                    <td>
                                        <input type="text" v-model="row.name" />
                                    </td>
                                    <td>
                                        <input type="number" v-model.number="row.amount" />
                                    </td>
                                </tr>
                                <tr>
                                    <td>Total</td>
                                    <td>
                                        <input type="number" :value="recoveryTotal" disabled />
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>

            <div class="statements-container">
                <div class="ubl-omni">
                    <table>
                        <tbody>
                            <tr>
                                <th colspan="2">UBL Omni</th>
                            </tr>

                            <tr>
                                <td>Balance</td>
                                <td>
                                    <input type="number" />
                                </td>
                            </tr>

                            <tr>
                                <td>Sending</td>
                                <td>Receiving</td>
                            </tr>

                            <tr v-for="index in 11" :key="index">
                                <td>
                                    <input type="text" />
                                </td>

                                <td>
                                    <input type="text" />
                                </td>
                            </tr>

                            <tr>
                                <td>Total Sending</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td>Total Receiving</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td>Last Balance</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>

                <div class="easypaisa">
                    <table>
                        <tbody>
                            <tr>
                                <th colspan="2">Easy Paisa</th>
                            </tr>

                            <tr>
                                <td>Balance</td>
                                <td>
                                    <input type="number" />
                                </td>
                            </tr>

                            <tr>
                                <td>Sending</td>
                                <td>Receiving</td>
                            </tr>

                            <tr v-for="index in 11" :key="index">
                                <td>
                                    <input type="text" />
                                </td>

                                <td>
                                    <input type="text" />
                                </td>
                            </tr>

                            <tr>
                                <td>Total Sending</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td>Total Receiving</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td>Last Balance</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>

                <div class="jazzcash">
                    <table>
                        <tbody>
                            <tr>
                                <th colspan="2">JazzCash</th>
                            </tr>

                            <tr>
                                <td>Balance</td>
                                <td>
                                    <input type="number" />
                                </td>
                            </tr>

                            <tr>
                                <td>Sending</td>
                                <td>Receiving</td>
                            </tr>

                            <tr v-for="index in 11" :key="index">
                                <td>
                                    <input type="text" />
                                </td>

                                <td>
                                    <input type="text" />
                                </td>
                            </tr>

                            <tr>
                                <td>Total Sending</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td>Total Receiving</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td>Last Balance</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>

                <div class="ep-account">
                    <table>
                        <tbody>
                            <tr>
                                <th colspan="2">EP Account 0333</th>
                            </tr>

                            <tr>
                                <td>Balance</td>
                                <td>
                                    <input type="number" />
                                </td>
                            </tr>

                            <tr>
                                <td>Sending</td>
                                <td>Receiving</td>
                            </tr>

                            <tr v-for="index in 11" :key="index">
                                <td>
                                    <input type="text" />
                                </td>

                                <td>
                                    <input type="text" />
                                </td>
                            </tr>

                            <tr>
                                <td>Total Sending</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td>Total Receiving</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td>Last Balance</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>

                <div class="jc-account">
                    <table>
                        <tbody>
                            <tr>
                                <th colspan="2">JC Account 0307</th>
                            </tr>

                            <tr>
                                <td>Balance</td>
                                <td>
                                    <input type="number" />
                                </td>
                            </tr>

                            <tr>
                                <td>Sending</td>
                                <td>Receiving</td>
                            </tr>

                            <tr v-for="index in 11" :key="index">
                                <td>
                                    <input type="text" />
                                </td>

                                <td>
                                    <input type="text" />
                                </td>
                            </tr>

                            <tr>
                                <td>Total Sending</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td>Total Receiving</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td>Last Balance</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>

                <div class="purchasing">
                    <table>
                        <tbody>
                            <tr>
                                <th colspan="2">Purchasing</th>
                            </tr>

                            <tr>
                                <td class="text-sm">UBL Omni Rec</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td class="text-sm">EasyPaisa Rec</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td class="text-sm">JazzCash Rec</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td class="text-sm">EP AC 0333 Rec</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td class="text-sm">JC AC 0307 Rec</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr v-for="index in 10" :key="index">
                                <td>
                                    <input type="text" />
                                </td>

                                <td>
                                    <input type="number" />
                                </td>
                            </tr>

                            <tr>
                                <td>Total</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>

            <div class="analytics-container">
                <div class="cash-detail">
                    <table>
                        <tbody>
                            <tr>
                                <th colspan="4">Cash Detail</th>
                            </tr>

                            <tr>
                                <td>5000</td>
                                <td>x</td>
                                <td><input type="number" /></td>
                                <td><input type="number" disabled /></td>
                            </tr>

                            <tr>
                                <td>1000</td>
                                <td>x</td>
                                <td><input type="number" /></td>
                                <td><input type="number" disabled /></td>
                            </tr>

                            <tr>
                                <td>500</td>
                                <td>x</td>
                                <td><input type="number" /></td>
                                <td><input type="number" disabled /></td>
                            </tr>

                            <tr>
                                <td>100</td>
                                <td>x</td>
                                <td><input type="number" /></td>
                                <td><input type="number" disabled /></td>
                            </tr>

                            <tr>
                                <td>50</td>
                                <td>x</td>
                                <td><input type="number" /></td>
                                <td><input type="number" disabled /></td>
                            </tr>

                            <tr>
                                <td>20</td>
                                <td>x</td>
                                <td><input type="number" /></td>
                                <td><input type="number" disabled /></td>
                            </tr>

                            <tr>
                                <td>10</td>
                                <td>x</td>
                                <td><input type="number" /></td>
                                <td><input type="number" disabled /></td>
                            </tr>

                            <tr>
                                <td>5</td>
                                <td>x</td>
                                <td><input type="number" /></td>
                                <td><input type="number" disabled /></td>
                            </tr>

                            <tr>
                                <td colspan="3">Total</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>

                <div class="analytics">
                    <table>
                        <tbody>
                            <tr>
                                <td>Previous Cash</td>
                                <td>
                                    <input type="number" />
                                </td>
                            </tr>

                            <tr>
                                <td>Today Cash</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td>Total Amount</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td>Recovery &amp; Purchasing</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td>Remaining Cash</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td>Difference</td>
                                <td>
                                    <input type="number" readonly />
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>

                <div class="red-book">
                    <table>
                        <tbody>
                            <tr>
                                <th colspan="2">RED Book</th>
                            </tr>

                            <tr v-for="index in 8" :key="index">
                                <td>
                                    <input type="text" />
                                </td>

                                <td>
                                    <input type="number" />
                                </td>
                            </tr>

                            <tr>
                                <td>Total</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>

                <div class="cash-info">
                    <table>
                        <tbody>
                            <tr>
                                <th colspan="2">Cash Info</th>
                            </tr>

                            <tr>
                                <td>UBL Omni</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td>EasyPaisa</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td>JazzCash</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td>EasyPaisa &amp; JazzCash Accounts</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td>Recovery</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td>Card</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td>EasyLoad</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>

                            <tr>
                                <td>Extra</td>
                                <td>
                                    <input type="number" />
                                </td>
                            </tr>

                            <tr>
                                <td>Total</td>
                                <td>
                                    <input type="number" disabled />
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
*,
*::before,
*::after {
    box-sizing: border-box;
}

body {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
        Oxygen, Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
    font-weight: bold;
}

table,
tr,
th,
td {
    border: 1px solid black;
}

input {
    width: 100px;
    color: black;
    text-align: center;
    font-weight: bold;
    font-size: 15px;
    border: none;
    height: 100%;
    outline: none;
}

input:disabled {
    background-color: #f0f0f0;
}

.mx {
    margin-left: 5px;
    margin-right: 5px;
}

.my {
    margin-top: 5px;
    margin-bottom: 5px;
}

.space-x {
    margin-left: 10px;
    margin-right: 10px;
}

.space-y {
    margin-top: 10px;
    margin-bottom: 10px;
}

.text-sm {
    font-size: 15px;
}

.text-lg {
    font-size: 25px;
}

.text-center {
    text-align: center;
}

.button {
    color: white;
    background-color: #1d4ed8;
    padding: 0.625rem 1.25rem;
    border-radius: 0.5rem;
    font-weight: 500;
    font-size: 0.875rem;
    margin-right: 0.5rem;
    margin-bottom: 0.5rem;
    transition: background-color 0.3s, box-shadow 0.3s;
    border: none;
    cursor: pointer;
}

.button:hover {
    background-color: #1e40af;
}

.button-danger {
    color: white;
    background-color: #c53030;
    padding: 0.625rem 1.25rem;
    border-radius: 0.5rem;
    font-weight: 500;
    font-size: 0.875rem;
    margin-right: 0.5rem;
    margin-bottom: 0.5rem;
    transition: background-color 0.3s, box-shadow 0.3s;
    border: none;
    cursor: pointer;
}

.button-danger:hover {
    background-color: #b91c1c;
}

.container {
    width: 100%;
    padding: 10px;
    margin: 0 auto;
}

.header {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 20px;
    margin-bottom: 20px;
}

.time-actions-container {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 50px;
}

.eload-accounts-container {
    display: flex;
    gap: 130px;
    align-items: start;
    justify-content: center;
    flex-wrap: wrap;
}

.easyload-container,
.accounts-container {
    display: flex;
    gap: 10px;
    align-items: start;
    justify-content: space-between;
    flex-wrap: wrap;
}

.borrow-recovery-container {
    display: flex;
    gap: 10px;
    flex-wrap: wrap;
}

.statements-container {
    display: flex;
    gap: 10px;
    align-items: start;
    justify-content: center;
    margin-bottom: 10px;
    flex-wrap: wrap;
}

.analytics-container {
    display: flex;
    gap: 82px;
    align-items: start;
    justify-content: center;
    flex-wrap: wrap;
}

input[type="number"]::-webkit-inner-spin-button,
input[type="number"]::-webkit-outer-spin-button {
    -webkit-appearance: none;
    -moz-appearance: none;
    appearance: none;
    margin: 0;
}
</style>
