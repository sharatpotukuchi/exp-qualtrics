# Treatment Assignment Fix - Step by Step Guide

**Issue:** `treatment_group` field is empty in data export  
**Root Cause:** Treatment assignment not properly configured in Qualtrics  
**Priority:** CRITICAL - Must fix before data collection

---

## 🔍 **Problem Analysis**

### **Current State:**
- ✅ JavaScript correctly reads `treatment_group: F(12)` from Loop & Merge
- ❌ Loop & Merge field 12 (`treatment_group`) is empty in scenario CSV
- ❌ No Randomizer block setting `treatment_group` as ED field
- ❌ Data export shows empty `treatment_group` values

### **Expected State:**
- ✅ Randomizer block sets `treatment_group` as ED field ("control" or "nudge")
- ✅ JavaScript reads from ED field, not Loop & Merge
- ✅ 50/50 split between control and nudge groups
- ✅ Data export shows populated `treatment_group` values

---

## 🛠️ **Solution Implementation**

### **Step 1: Fix Qualtrics Randomizer Block**

#### **1.1 Create Randomizer Block**
1. **Go to Survey Flow** in Qualtrics
2. **Add Randomizer Block** before the Loop & Merge block
3. **Configure Randomizer:**
   - **Randomization Type:** Simple Randomization
   - **Percentages:** 50% / 50%
   - **Number of Elements:** 2

#### **1.2 Configure Randomizer Elements**
**Element 1 (Control Group - 50%):**
- **Type:** Embedded Data
- **Field Name:** `treatment_group`
- **Value:** `control`

**Element 2 (Nudge Group - 50%):**
- **Type:** Embedded Data  
- **Field Name:** `treatment_group`
- **Value:** `nudge`

#### **1.3 Add Treatment Guard (Backup)**
Add a second Randomizer block as backup:
- **Field Name:** `treatment_guard`
- **Same 50/50 split**
- **Values:** `control` / `nudge`

### **Step 2: Update JavaScript Code**

#### **2.1 Modify Treatment Assignment Logic**
Replace the current Loop & Merge approach with ED field reading:

```javascript
// OLD CODE (Remove this):
treatment_group: F(12),

// NEW CODE (Add this):
// Read treatment_group from ED field instead of Loop & Merge
treatment_group: Qualtrics.SurveyEngine.getEmbeddedData('treatment_group') || 'control',
```

#### **2.2 Add Treatment Validation**
Add validation to ensure treatment assignment is working:

```javascript
// Add treatment validation
var treatmentGroup = Qualtrics.SurveyEngine.getEmbeddedData('treatment_group');
if (!treatmentGroup || (treatmentGroup !== 'control' && treatmentGroup !== 'nudge')) {
  console.error('Treatment assignment failed:', treatmentGroup);
  // Fallback to control
  Qualtrics.SurveyEngine.setEmbeddedData('treatment_group', 'control');
  treatmentGroup = 'control';
}
```

#### **2.3 Add Order Permission Logic**
Add `buy_allowed` and `sell_allowed` based on `allowed_sides`:

```javascript
// Add order permission logic
var allowedSides = Qualtrics.SurveyEngine.getEmbeddedData('allowed_sides') || 'buy_sell';
var buyAllowed = (allowedSides === 'buy_sell' || allowedSides === 'buy_only') ? 1 : 0;
var sellAllowed = (allowedSides === 'buy_sell' || allowedSides === 'sell_only') ? 1 : 0;

Qualtrics.SurveyEngine.setEmbeddedData('buy_allowed', buyAllowed);
Qualtrics.SurveyEngine.setEmbeddedData('sell_allowed', sellAllowed);
```

### **Step 3: Update Scenario CSV**

#### **3.1 Remove treatment_group from CSV**
Since treatment assignment is now handled by Randomizer, remove it from the scenario CSV:
- **Remove column 12** (`treatment_group`) from `scenario_pack_phase1_v6_enhanced_max_effect.csv`
- **Update JavaScript mapping** to shift field numbers

#### **3.2 Update JavaScript Field Mapping**
After removing `treatment_group` from CSV, update the field mapping:

```javascript
var map = {
  scenario_id: F(1),
  scenario_name: F(2),
  symbol: F(3),
  prices_json: F(4),
  bid: F(5),
  ask: F(6),
  last: F(7),
  balance: F(8),
  pos_qty: F(9),
  pos_px: F(10),
  hot_condition: F(11),
  // treatment_group: F(12), // REMOVED - now from ED field
  sentiment_pct: F(12), // Shifted up
  session_tag: F(13),   // Shifted up
  // ... continue shifting all fields up by 1
};
```

### **Step 4: Test Treatment Assignment**

#### **4.1 Preview Mode Testing**
1. **Test in Qualtrics Preview Mode**
2. **Complete survey 10 times**
3. **Verify 50/50 split** in data export
4. **Check ED field population**

#### **4.2 Data Export Validation**
Verify the following in data export:
- ✅ `treatment_group` shows "control" or "nudge"
- ✅ `treatment_guard` shows "control" or "nudge"  
- ✅ `buy_allowed` and `sell_allowed` populated correctly
- ✅ Roughly 50/50 split between groups

---

## 📋 **Implementation Checklist**

### **Qualtrics Configuration:**
- [ ] Create Randomizer block before Loop & Merge
- [ ] Configure 50/50 split between control/nudge
- [ ] Set `treatment_group` ED field in Randomizer
- [ ] Add `treatment_guard` backup Randomizer
- [ ] Test Randomizer in preview mode

### **JavaScript Updates:**
- [ ] Replace `treatment_group: F(12)` with ED field reading
- [ ] Add treatment validation logic
- [ ] Add order permission logic (`buy_allowed`, `sell_allowed`)
- [ ] Update field mapping after CSV changes
- [ ] Test JavaScript changes

### **CSV Updates:**
- [ ] Remove `treatment_group` column from scenario CSV
- [ ] Update JavaScript field mapping
- [ ] Test with updated CSV

### **Testing:**
- [ ] Test treatment assignment in preview mode
- [ ] Verify 50/50 split in test data
- [ ] Check data export quality
- [ ] Validate ED field population

---

## 🚨 **Critical Success Factors**

### **Must Have:**
1. **Randomizer block working** (50/50 split)
2. **ED fields populated** (`treatment_group`, `treatment_guard`)
3. **JavaScript reading from ED fields** (not Loop & Merge)
4. **Order permissions set** (`buy_allowed`, `sell_allowed`)

### **Validation Tests:**
1. **Preview Mode Test:** Complete survey 10 times, check 50/50 split
2. **Data Export Test:** Verify all treatment fields populated
3. **JavaScript Test:** Check console for treatment assignment errors

---

## 📊 **Expected Results**

### **Before Fix:**
```csv
treatment_group,treatment_guard,buy_allowed,sell_allowed
,,,
,,,
```

### **After Fix:**
```csv
treatment_group,treatment_guard,buy_allowed,sell_allowed
control,control,1,1
nudge,nudge,1,1
control,control,1,1
nudge,nudge,1,1
```

---

**Fix Created By:** AI Assistant  
**Date:** October 2, 2025  
**Priority:** CRITICAL  
**Estimated Time:** 2 days  
**Next Review:** After implementation complete
