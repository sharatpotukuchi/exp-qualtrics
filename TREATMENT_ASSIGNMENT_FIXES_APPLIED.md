# Treatment Assignment Fixes - Applied Corrections

**Status:** Issues identified - fixes need to be applied to JavaScript code

---

## 🔍 **Current Issues Found**

### **1. Loop & Merge Field Mapping (Line 38)**
```javascript
// ❌ WRONG - Still reading from empty Loop & Merge field
treatment_group: F(12),
```

### **2. Missing sell_allowed Field (Line 1411)**
```javascript
// ❌ INCOMPLETE - Only buy_allowed is set
Qualtrics.SurveyEngine.setEmbeddedData('buy_allowed', buyAllowed ? '1' : '0');
```

### **3. Complex Retry Logic**
The current retry logic is overly complex and may not be necessary if Randomizer is properly configured.

---

## 🛠️ **Required JavaScript Fixes**

### **Fix 1: Remove treatment_group from Loop & Merge Mapping**
```javascript
// OLD CODE (Line 38):
treatment_group: F(12),

// NEW CODE:
// treatment_group: F(12), // REMOVED - now handled by Randomizer
```

### **Fix 2: Add sell_allowed Field**
```javascript
// ADD AFTER Line 1411:
try {
  Qualtrics.SurveyEngine.setEmbeddedData('buy_allowed', buyAllowed ? '1' : '0');
  Qualtrics.SurveyEngine.setEmbeddedData('sell_allowed', sellAllowed ? '1' : '0');
} catch (e) {}
```

### **Fix 3: Simplify Treatment Assignment Logic**
```javascript
// REPLACE Lines 1614-1648 with:
// Read treatment_group from Randomizer ED field
var treatmentGroupValue = ED('treatment_group') || '';
var forceNudge = /[?&]nudge=1(&|$)/.test(location.search) || ED('debug_force_nudge') === '1';
var shouldShowNudge = (cleanAlpha(treatmentGroupValue) === 'nudge' || 
                      cleanAlpha(treatmentGroupValue) === 'generic_nudge') || forceNudge;

// Set treatment guard backup
try {
  Qualtrics.SurveyEngine.setEmbeddedData('treatment_guard', 
    shouldShowNudge ? 'nudge' : 'control');
} catch (e) {}

console.log('TREATMENT →', { 
  treatmentGroupValue: treatmentGroupValue, 
  shouldShowNudge: shouldShowNudge
});
```

---

## 📋 **Implementation Checklist**

### **JavaScript Changes:**
- [ ] Remove `treatment_group: F(12)` from Loop & Merge mapping
- [ ] Add `sell_allowed` ED field setting
- [ ] Simplify treatment assignment logic
- [ ] Test treatment assignment in preview mode

### **Qualtrics Configuration:**
- [ ] Verify Randomizer block is configured before Loop & Merge
- [ ] Set `treatment_group` ED field in Randomizer (50/50 split)
- [ ] Add `treatment_guard` backup Randomizer
- [ ] Test Randomizer in preview mode

### **CSV Updates:**
- [ ] Remove `treatment_group` column from scenario CSV
- [ ] Update JavaScript field mapping (shift all fields up by 1)
- [ ] Test with updated CSV

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

## 📊 **Expected Results After Fix**

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

**Status:** Ready for implementation  
**Priority:** CRITICAL  
**Estimated Time:** 1 day  
**Next Review:** After JavaScript changes applied
