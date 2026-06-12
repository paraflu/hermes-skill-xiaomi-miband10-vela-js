# Device Storage Reservation

# 设备存储空间预留数据（Device Storage Reservation）

> 来源：com.bandbbs.ebook 项目 `src/utils/storageUtils.js`
> 官方文档状态：**官方未收录** — 官方有 `device.getTotalStorage()` 和 `device.getAvailableStorage()` 接口，但未说明各设备的系统预留空间

## 背景

IoT 设备的存储空间并非全部可供应用使用，系统会预留一部分空间用于固件、日志等。不同设备的预留空间差异很大，直接使用 `getAvailableStorage()` 返回值可能不准确。

参考项目维护了一张**各设备型号的系统预留空间表**，用于计算实际可用空间。

## 完整代码

```javascript
/**
 * 获取指定设备的系统预留存储空间（字节）
 * @param {string} product - 设备型号，来自 device.getInfo().product
 * @returns {number} 预留空间（字节）
 */
export function getReservedStorage(product) {
  if (!product) return 0

  // Redmi Watch 系列：120MB
  if (product === 'REDMI Watch 6') return 120 * 1024 * 1024
  if (product === 'REDMI Watch 5') return 120 * 1024 * 1024

  // 小米手环 9 系列：64MB
  if (product === 'Xiaomi Smart Band 9') return 64 * 1024 * 1024
  if (product === 'Xiaomi Smart Band 9 Pro') return 64 * 1024 * 1024

  // 小米手环 8 Pro：84MB
  if (product === 'Xiaomi Smart Band 8 Pro') return 84 * 1024 * 1024

  // 小米手环 10 系列：90MB
  if (product && product.includes('Xiaomi Smart Band 10')) return 90 * 1024 * 1024

  // o65m（开发代号）：1GB
  if (product === 'o65m') return 1024 * 1024 * 1024

  // 未知设备：不预留
  return 0
}

/**
 * 计算存储空间详情
 * @param {number} totalStorage - 总存储空间（字节），来自 device.getTotalStorage()
 * @param {number} availableStorage - 可用存储空间（字节），来自 device.getAvailableStorage()
 * @param {string} product - 设备型号
 * @returns {Object} 存储空间详情
 */
export function calculateStorageInfo(totalStorage, availableStorage, product) {
  const reservedStorage = getReservedStorage(product)
  const usedStorage = totalStorage - availableStorage
  const actualAvailable = totalStorage - reservedStorage - usedStorage

  return {
    totalStorage,        // 总空间
    availableStorage,    // 系统报告可用空间
    reservedStorage,     // 系统预留空间
    usedStorage,         // 已使用空间
    actualAvailable      // 实际可供应用使用的空间
  }
}

/**
 * 判断存储空间是否不足
 * @param {number} actualAvailable - 实际可用空间（字节）
 * @returns {boolean} 是否空间不足
 */
export function isStorageLow(actualAvailable) {
  return actualAvailable < 2 * 1024 * 1024 // 低于 2MB 视为不足
}
```

## 使用示例

```javascript
import device from '@system.device'
import { calculateStorageInfo, isStorageLow } from '../../utils/storageUtils'

export default {
  data: {
    storageInfo: {},
    storageLow: false
  },
  onInit() {
    device.getInfo({
      success: (info) => {
        const product = info.product
        device.getTotalStorage({
          success: (total) => {
            device.getAvailableStorage({
              success: (avail) => {
                this.storageInfo = calculateStorageInfo(
                  total.totalStorage,
                  avail.availableStorage,
                  product
                )
                this.storageLow = isStorageLow(this.storageInfo.actualAvailable)

                if (this.storageLow) {
                  prompt.showToast({
                    message: '存储空间不足，请清理文件',
                    duration: 3000
                  })
                }
              }
            })
          }
        })
      }
    })
  }
}
```

## 已知设备存储规格

| 设备型号 | 系统预留 | 备注 |
|----------|----------|------|
| REDMI Watch 5 | 120 MB | |
| REDMI Watch 6 | 120 MB | |
| Xiaomi Smart Band 8 Pro | 84 MB | |
| Xiaomi Smart Band 9 | 64 MB | |
| Xiaomi Smart Band 9 Pro | 64 MB | |
| Xiaomi Smart Band 10 | 90 MB | 包含 Pro 变体 |

## 重要提醒

1. 此数据基于实际设备测试，固件更新可能导致数值变化
2. 新设备发布后需要补充对应数据
3. `getAvailableStorage()` 返回值已扣除系统使用部分，但未扣除系统预留部分
4. 建议在应用启动时检查存储空间，空间不足时提示用户
