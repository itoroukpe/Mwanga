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

The key insight is that the Holy Calendar runs on pure 360-day years with no leap days, so it "runs faster" than the Gregorian calendar (which averages 365.2425 days/year). Over ~2000+ years, this creates a 34-year offset by 2020, which is why the Holy year is 2054.

Implement the plan
