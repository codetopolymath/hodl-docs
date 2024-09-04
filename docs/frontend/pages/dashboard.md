# Dashboard Component Code-Level Documentation

## Component Overview

The Dashboard component is a complex React component that displays various user statistics, staking information, and project history. It integrates with Web3 technologies and interacts with smart contracts.

## Import Statements

```javascript
import React, { useEffect, useState } from "react";
import { useWeb3ModalAccount, useWeb3ModalProvider } from "@web3modal/ethers/react";
import { BigNumberish, formatEther, formatUnits, parseEther } from "ethers";
import { Area, AreaChart, ResponsiveContainer, XAxis } from "recharts";
// ... other imports
```

- The component uses React hooks for state management and side effects.
- Web3Modal hooks are used for blockchain interactions.
- Ethers.js is used for handling blockchain data types and conversions.
- Recharts is used for rendering the staking overview chart.

## Main Component Function

```javascript
export default function Dashboard() {
  // ... component logic
}
```

The Dashboard is exported as the default function component.

## State Declarations

```javascript
const [data, setData] = useState<any>();
const [chart, setChart] = useState<any>();
const [nftHistory, setNftHistory] = useState<any[]>([]);
const [hasUSDTAllowance, setHasUSDTAllowance] = useState(false);
```

- `data`: Stores the main dashboard data fetched from the API.
- `chart`: Stores processed data for the staking overview chart.
- `nftHistory`: Stores the user's NFT transaction history.
- `hasUSDTAllowance`: Boolean flag indicating if the user has given USDT allowance for staking.

## Web3 Hooks

```javascript
const { address, chainId } = useWeb3ModalAccount();
const { walletProvider } = useWeb3ModalProvider();
```

These hooks provide the user's wallet address, connected chain ID, and wallet provider for blockchain interactions.

## Data Fetching Functions

### `getData()`

```javascript
const getData = async () => {
  try {
    const incufiContract = await getIncufiContract(walletProvider);
    const t12 = await incufiContract?.commisionAmount(address);

    const akitaDContract = await getAkitaDContract(walletProvider);
    const t13 = await akitaDContract?.balanceOf(address);

    const res = await axiosAuthClient.get("user/dashboard/");
    setData({
      ...res.data,
      totalCommission: Number(formatEther(t12)),
      commissionBalance: Number(formatEther(t13)),
    });
  } catch (error) {
    console.log(error);
  }
};
```

This function fetches dashboard data from both the blockchain (using smart contracts) and the backend API.

### `getNftHistory()`

```javascript
const getNftHistory = async () => {
  try {
    const res = await axiosAuthClient.get("akita/nfts/");
    setNftHistory(res.data);
  } catch (error) {
    console.log(error);
  }
};
```

Fetches the user's NFT transaction history from the backend API.

## useEffect Hooks

```javascript
useEffect(() => {
  if (data) {
    const temp = data?.stake_dates.map((date: any, index: number) => {
      return { name: formatDate(date), value: data?.stake_amounts[index] };
    });
    setChart(temp);
  }
}, [data]);

useEffect(() => {
  if (walletProvider && chainId) getData();
}, [walletProvider, chainId]);

useEffect(() => {
  getNftHistory();
}, []);

useEffect(() => {
  if (walletProvider && address) {
    checkUSDTAllowance();
  }
}, [walletProvider, address, data]);
```

These effects handle:
1. Processing chart data when the main data is fetched.
2. Fetching dashboard data when the wallet is connected.
3. Fetching NFT history on component mount.
4. Checking USDT allowance when the wallet is connected or data changes.

## Blockchain Interaction Functions

### `withdrawCommission()`

```javascript
const withdrawCommission = async () => {
  try {
    const res = await axiosAuthClient.post("incufi/withdraw-commission/");
    Swal.fire({
      title: "Success",
      text: "Withdraw request placed successfully",
      icon: "success",
    }).then((res) => router.push("withdraw-history"));
  } catch (error: any) {
    Swal.fire({
      title: "Error",
      text: `${error?.response.data[0]}`,
      icon: "error",
    });
    console.log(error);
  }
};
```

Handles the commission withdrawal process, interacting with the backend API and providing user feedback.

### `stakeForNFT()`

```javascript
const stakeForNFT = async () => {
  if (data?.stake_remaining == 0) {
    Swal.fire({
      title: "Error",
      text: `You dont have any additional amount`,
      icon: "error",
    });
    return;
  }
  if (hasUSDTAllowance) {
    try {
      const purchase = await getPurchaseContract(walletProvider);
      const trans = await purchase?.Invest(
        parseEther((100 - data?.stake_remaining).toString())
      );
      const receipt = await trans.wait();
      const res = await axiosAuthClient.post("incufi/stake-remaining/", {
        amount: 100 - data?.stake_remaining,
        transaction_hash: receipt.hash,
        staking_duration: 90,
      });
      Swal.fire({
        title: "Success",
        text: "Amount staked successfully",
        icon: "success",
      });
      window.location.reload();
    } catch (error: any) {
      console.log(error);
      Swal.fire({
        title: "Error",
        text: `${error.reason}`,
        icon: "error",
      });
    }
  } else {
    takeUSDTAllowance();
  }
};
```

Handles the staking process for NFT acquisition, including smart contract interaction and backend API calls.

## Subcomponents

### StatCard

```javascript
const StatCard = ({
  title,
  value,
  unit,
}: {
  title: any;
  value: any;
  unit: any;
}) => (
  <div className={`${styles.statCard} mb-3`}>
    <h3>{title}</h3>
    <p className={styles.statValue}>
      {value} {unit}
    </p>
  </div>
);
```

A reusable component for displaying individual statistic cards.

### CircularProgress

```javascript
const CircularProgress = ({
  percentage,
  size = 60,
}: {
  percentage: any;
  size: any;
}) => (
  <div
    className={styles.circularProgress}
    style={{ width: size, height: size }}
  >
    {/* SVG implementation */}
  </div>
);
```

A custom component for rendering circular progress indicators.

## Render Method

The render method of the Dashboard component is extensive, utilizing conditional rendering and mapping over data arrays to display various sections:

1. Staking Overview Chart
2. Statistic Cards
3. Team Referrals
4. Project History

Each section is wrapped in appropriate CSS classes for styling and layout.

## CSS Modules

The component uses CSS modules for styling, importing styles from `dashboard.module.css`:

```javascript
import styles from "./dashboard.module.css";
```

This allows for scoped styling and prevents class name conflicts.

## Performance Considerations

- The component uses `useEffect` hooks to manage side effects and prevent unnecessary re-renders.
- Conditional rendering is used to display data only when it's available.
- The staking chart uses `ResponsiveContainer` from Recharts for responsive rendering.

## Error Handling

Error handling is implemented in data fetching functions and blockchain interactions, typically logging errors to the console and displaying user-friendly messages using SweetAlert2.

## Conclusion

The Dashboard component is a complex piece of UI that integrates various data sources, blockchain interactions, and visualizations. It demonstrates advanced React patterns, Web3 integration, and responsive design principles.