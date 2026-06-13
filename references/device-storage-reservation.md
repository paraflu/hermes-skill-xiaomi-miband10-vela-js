# Device Storage Reservation

## Device storage reservation data

> Source: com.bandbbs.ebook project `src/utils/storageUtils.js`
> Official documentation status: **Not listed in official docs** — APIs `device.getTotalStorage()` and `device.getAvailableStorage()` exist, but official docs do not specify system reserved space per device

## Background

Not all device storage is available to applications; the system reserves space for firmware, logs, etc. Reserved amounts vary across devices, so `getAvailableStorage()` may be misleading.

The reference project maintains a **table of system reserved space by device model** to compute actual available space.

## Full Code

```javascript
/**
 * Get system reserved storage for a given device (bytes)
 * @param {string} product - device model from device.getInfo().product
 * @returns {number} reserved space (bytes)
 */
export function getReservedStorage(product) {
  if (!product) return 0

  // Redmi Watch series: 120MB
  if (product === 'REDMI Watch 6') return 120 * 1024 * 1024
  if (product === 'REDMI Watch 5') return 120 * 1024 * 1024

  // Xiaomi Smart Band 9 series: 64MB
  if (product === 'Xiaomi Smart Band 9') return 64 * 1024 * 1024
  if (product === 'Xiaomi Smart Band 9 Pro') return 64 * 1024 * 1024

  // Xiaomi Smart Band 8 Pro: 84MB
  if (product === 'Xiaomi Smart Band 8 Pro') return 84 * 1024 * 1024

  // Xiaomi Smart Band 10 series: 90MB
  if (product && product.includes('Xiaomi Smart Band 10')) return 90 * 1024 * 1024

  // o65m (codename): 1GB
  if (product === 'o65m') return 1024 * 1024 * 1024

  // Unknown devices: no reservation
  return 0
}

/**
 * Calculate storage information
 * @param {number} totalStorage - total storage (bytes) from device.getTotalStorage()
 * @param {number} availableStorage - available storage (bytes) from device.getAvailableStorage()
 * @param {string} product - device model
 * @returns {Object} storage details
 */
export function calculateStorageInfo(totalStorage, availableStorage, product) {
  const reservedStorage = getReservedStorage(product)
  const usedStorage = totalStorage - availableStorage
  const actualAvailable = totalStorage - reservedStorage - usedStorage

  return {
    totalStorage,        // total space
    availableStorage,    // system-reported available space
    reservedStorage,     // system reserved space
    usedStorage,         // used space
    actualAvailable      // actual available space for the app
  }
}

/**
 * Determine if storage is low
 * @param {number} actualAvailable - actual available space (bytes)
 * @returns {boolean} whether storage is low
 */
export function isStorageLow(actualAvailable) {
  return actualAvailable < 2 * 1024 * 1024 // consider low if below 2MB
}
```

## Usage Example

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
                    message: 'Insufficient storage, please clean up files',
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

## Known Device Storage Specifications

| Device Model | System Reserved | Notes |
|----------|----------|------|
| REDMI Watch 5 | 120 MB | |
| REDMI Watch 6 | 120 MB | |
| Xiaomi Smart Band 8 Pro | 84 MB | |
| Xiaomi Smart Band 9 | 64 MB | |
| Xiaomi Smart Band 9 Pro | 64 MB | |
| Xiaomi Smart Band 10 | 90 MB | Includes Pro variant |

## Important Notes

1. This data is based on actual device testing; firmware updates may change these values
2. Data must be updated when new devices are released
3. The `getAvailableStorage()` return value has deducted system usage but has **not** deducted the system reserved space
4. It is recommended to check storage space when the app starts and prompt the user if space is low
