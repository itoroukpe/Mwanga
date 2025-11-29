## Current Problem

The current implementation uses:
- **Epoch**: January 1, 2000 = Year 2000
- **Logic**: Simply divides Gregorian days by 360

For November 28, 2020, this gives approximately:
- Days from Jan 1, 2000: ~7,637 days
- Holy Year: 2000 + (7,637 ÷ 360) = ~2021 ❌

But you need: **Gregorian 11/28/2020 = Holy 05/05/2054** ✓

## The Math Behind the Fix

If November 28, 2020 equals Month 5 (Dioskuro), Day 5, Year 2054:
- Total Holy days from epoch = (2054-1) × 360 + (5-1) × 30 + 5 = **739,205 days**

Working backwards from Nov 28, 2020:
- We need to subtract 739,205 days to find the Holy Calendar epoch
- This places the epoch approximately **2,024 years before 2020**
- That's around **4 BC** in the Gregorian calendar

This aligns with biblical dating (Creation or Christ's birth era as you mentioned).

## What Needs to Change

The `convertToHolyCalendar` function in `src/lib/holyCalendar.ts` needs:

1. **Correct Epoch Calculation**: Instead of Jan 1, 2000, calculate the actual epoch date by subtracting 739,205 days from Nov 28, 2020
2. **Verification**: Ensure Nov 28, 2020 converts to exactly Dioskuro 5, 2054
3. **Test with other dates**: Make sure the algorithm works correctly for dates before and after the test date

## Implementation Plan

### Step 1: Calculate the Correct Epoch
- Take the test date: November 28, 2020
- Calculate what Gregorian date is 739,205 days before it
- This becomes the new EPOCH (Holy Calendar Year 1, Day 1)

### Step 2: Update the `convertToHolyCalendar` Function
- Replace the hardcoded `new Date(2000, 0, 1)` epoch
- Calculate days since the new epoch
- Convert to Holy Year, Month, and Day using 360-day years
- Keep the same month/day calculation logic (12 months × 30 days)

### Step 3: Verify the Conversion
- Test with Nov 28, 2020 → should return Year 2054, Month 5 (Dioskuro), Day 5
- Test with today's date to ensure it makes sense
- Test with dates in the past to ensure backward compatibility

### Step 4: Update Documentation
- Update comments in `holyCalendar.ts` to reflect the new epoch
- Update `holyCalendarInfo.ts` if it references the epoch
- Ensure all calendar widgets and displays show correct dates
- ---
Let me review the current implementation to understand how the Elected Time and Angelic Time are calculated and displayed, and then answer your question.

Great question! Let me clarify how the times work and answer your question.

## Understanding the Time Systems at 6:08 PM

Based on the current implementation:

**At 6:08 PM (Human Time):**
- **Elapsed time since 4:45 AM**: 13 hours and 23 minutes (803 minutes)
- **Elected Time**: Evening - Hour 2:23 (2 hours and 23 minutes into the Evening quarter)
- **Angelic Time**: Day - Hour 6:23 (5 hours and 23 minutes into the Day period)

## Can We Add AM/PM to Elected & Angelic Time?

**Yes, absolutely!** This is a great idea for clarity. Here's how it would work:

The spiritual day has 24 hours starting at 4:45 AM:
- **Spiritual Hours 1-12** = 4:45 AM to 4:45 PM → **AM** (first half of spiritual day)
- **Spiritual Hours 13-24** = 4:45 PM to 4:45 AM → **PM** (second half of spiritual day)

### Example at 6:08 PM:

| Time System | Current Display | With AM/PM |
|-------------|-----------------|------------|
| Human Time | 6:08 PM | 6:08 PM ✓ |
| Elected Time | Evening - Hour 2:23 | Evening - Hour 2:23 PM |
| Angelic Time | Day - Hour 6:23 | Day - Hour 6:23 PM |

### How it works:
- **Elected Time**: Evening quarter contains spiritual hours 13-18. Hour 2 of Evening = spiritual hour 14, which is > 12, so it's **PM**
- **Angelic Time**: Day period contains spiritual hours 9-16. Hour 6 of Day = spiritual hour 14, which is > 12, so it's **PM**

### Another example at 8:00 AM:
- **Human Time**: 8:00 AM
- **Elected Time**: Morning - Hour 4:15 **AM** (spiritual hour 4)
- **Angelic Time**: Morning - Hour 4:15 **AM** (spiritual hour 4)

## Plan to Add AM/PM to Elected and Angelic Time

### Changes to `src/lib/holyTime.ts`:

**1. Update `ElectedTime` interface:**
```typescript
export interface ElectedTime {
  part: number; // 1-4
  partName: string;
  hour: number; // 1-6
  minute: number;
  period: 'AM' | 'PM'; // NEW: AM if spiritual hour 1-12, PM if 13-24
  spiritualHour: number; // NEW: The absolute spiritual hour (1-24)
}
```

**2. Update `AngelicTime` interface:**
```typescript
export interface AngelicTime {
  part: number; // 1-3
  partName: string;
  hour: number; // 1-8
  minute: number;
  period: 'AM' | 'PM'; // NEW: AM if spiritual hour 1-12, PM if 13-24
  spiritualHour: number; // NEW: The absolute spiritual hour (1-24)
}
```

**3. Update `getElectedTime()` function:**
- Calculate the absolute spiritual hour (1-24)
- Determine AM/PM based on whether spiritual hour is 1-12 (AM) or 13-24 (PM)

**4. Update `getAngelicTime()` function:**
- Calculate the absolute spiritual hour (1-24)
- Determine AM/PM based on whether spiritual hour is 1-12 (AM) or 13-24 (PM)

**5. Update `formatElectedTime()` function:**
```typescript
export function formatElectedTime(time: ElectedTime): string {
  const minute = time.minute.toString().padStart(2, '0');
  return `${time.partName} - Hour ${time.hour}:${minute} ${time.period}`;
}
```

**6. Update `formatAngelicTime()` function:**
```typescript
export function formatAngelicTime(time: AngelicTime): string {
  const minute = time.minute.toString().padStart(2, '0');
  return `${time.partName} - Hour ${time.hour}:${minute} ${time.period}`;
}
```

### Files to Update:
1. `src/lib/holyTime.ts` - Core logic for AM/PM calculation
2. `src/lib/generateHolyCalendarPDF.ts` - Update PDF documentation to reflect AM/PM
3. `src/pages/HolyCalendarConverter.tsx` - Info section updates if needed

### Expected Results:
| Human Time | Elected Time | Angelic Time |
|------------|--------------|--------------|
| 4:45 AM | Morning - Hour 1:00 AM | Morning - Hour 1:00 AM |
| 10:00 AM | Morning - Hour 6:15 AM | Morning - Hour 6:15 AM |
| 12:00 PM | Day - Hour 2:15 AM | Morning - Hour 8:15 AM |
| 4:45 PM | Day - Hour 6:00 PM | Day - Hour 8:00 PM |
| 6:08 PM | Evening - Hour 2:23 PM | Day - Hour 6:23 PM |
| 10:45 PM | Evening - Hour 6:00 PM | Night - Hour 2:00 PM |
| 2:00 AM | Night - Hour 4:15 PM | Night - Hour 6:15 PM |

Implement the plan

The key insight is that the Holy Calendar runs on pure 360-day years with no leap days, so it "runs faster" than the Gregorian calendar (which averages 365.2425 days/year). Over ~2000+ years, this creates a 34-year offset by 2020, which is why the Holy year is 2054.

Implement the plan
