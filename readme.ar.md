<div dir=rtl>

# لـاما - روابط الأسس لمكتبة llama.cpp
[[English]](readme.md)

روابط لغة الأسس لمكتبة [llama.cpp](https://github.com/ggerganov/llama.cpp)، توفر واجهة كاملة لتشغيل استدلال نماذج اللغة الكبيرة محلياً. تدعم هذه المكتبة المعالج المركزي ومعالج الرسوميات عبر Vulkan.

## التثبيت

```
اشمل "مـحا"؛
مـحا.اشمل_ملف("Alusus/Llama"، "لـاما.أسس")؛
```

<div dir=ltr>

```
import "Apm";
Apm.importFile("Alusus/Llama");
```

</div>

## الاستخدام

```
اشمل "مـتم/طـرفية"؛
اشمل "مـحا"؛
مـحا.اشمل_ملف("Alusus/Llama"، "لـاما.أسس")؛

// تحميل المشغلات
جـجمل.مـشغل.حمل_للمعالج()؛
جـجمل.مـشغل.حمل_لفلكان()؛  // اختياري: لدعم Vulkan

// تحميل النموذج
عرف نموذج: سند[نـموذج](نـموذج.حمل("model.gguf"، نـموذج.هات_معطيات_افتراضية()))؛
عرف سياق: سند[سـياق](سـياق.هيئ_من_نموذج(نموذج، سـياق.هات_معطيات_افتراضية()))؛

// ترميز المدخلات
عرف رموز: مصفوفة[رمـز، 512]؛
عرف عدد_الرموز: صـحيح = رمز(نموذج.المفردات، "مرحبا"، 10، رموز، 512، 1، 1)؛

// فك الترميز والتوليد
عرف دفعة: دفـعة = دفـعة.هات_واحدا(رموز، عدد_الرموز)؛
سياق.فك_ترميز(دفعة)؛

// معاينة الرمز التالي
عرف معاين: سند[مـعاين](مـعاين.هيئ_جشع())؛
عرف الرمز_التالي: رمـز = معاين.عاين(سياق، -1)؛

// التنظيف
مـعاين.حرر(معاين)؛
سـياق.حرر(سياق)؛
نـموذج.حرر(نموذج)؛
```

<div dir=ltr>

```
import "Srl/Console";
import "Apm";
Apm.importFile("Alusus/Llama");
use Srl;
use Llama;

// Load backends
Ggml.Backend.cpuLoad();
Ggml.Backend.vkLoad();  // Optional: for Vulkan GPU support

// Load model
def model: ref[Model](Model.load("model.gguf", Model.getDefaultParams()));
def ctx: ref[Context](Context.initFromModel(model, Context.getDefaultParams()));

// Tokenize input
def tokens: array[Token, 512];
def nTokens: Int = tokenize(model.vocab, "Hello", 5, tokens, 512, true, true);

// Decode and generate
def batch: Batch = Batch.getOne(tokens, nTokens);
ctx.decode(batch);

// Sample next token
def sampler: ref[Sampler](Sampler.initGreedy());
def nextToken: Token = sampler.sample(ctx, -1);

// Cleanup
Sampler.free(sampler);
Context.free(ctx);
Model.free(model);
```

</div>

## دعم معالج الرسوميات

حدد متغير البيئة `GGML_USE_VULKAN` بقيمة `1` قبل التشغيل لتفعيل تسريع Vulkan:

<div dir=ltr>

```bash
GGML_USE_VULKAN=1 alusus your_script.alusus
```

</div>

---

## مرجع الواجهة البرمجية

### الأنواع الأساسية

| النوع بالعربية | النوع بالإنجليزية          | نوع llama.cpp      | الوصف                                |
|----------------|---------------------------|--------------------|--------------------------------------|
| `رمـز`         | `Token` |             `llama_token`            | معرف الرمز (لقب لـ `صحيح` / `Int`)   |
| `مـوقع`        | `Pos` |               `llama_pos`              | موقع الرمز (لقب لـ `صحيح` / `Int`)   |
| `مـعرف_تسلسل`  | `SeqId` |            `llama_seq_id`            | معرف التسلسل (لقب لـ `صحيح` / `Int`) |
| `دالـة_تقدم`   | `ProgressCallback` | `llama_progress_callback` | مؤشر دالة التقدم                     |

### الثوابت

| الثابت بالعربية      | الثابت بالإنجليزية | ثابت llama.cpp       | القيمة         | الوصف                      |
|----------------------|-------------------|----------------------|----------------|----------------------------|
| `_البذرة_الافتراضية_` | `DEFAULT_SEED` | `LLAMA_DEFAULT_SEED` |      `0xFFFFFFFF` | البذرة العشوائية الافتراضية |
| `_رمز_عدمي_`         | `TOKEN_NULL` |   `LLAMA_TOKEN_NULL` |              `-1`   | قيمة الرمز العدمي          |

---

## السرود

### نـمط_تقسيم / SplitMode

يتحكم في كيفية توزيع طبقات النموذج على معالجات الرسوميات.

| القيمة بالعربية | القيمة بالإنجليزية        | قيمة llama.cpp | الوصف                                  |
|-----------------|--------------------------|----------------|----------------------------------------|
| `_لا_شيء_`       | `NONE` |  `LLAMA_SPLIT_MODE_NONE`         | معالج رسوميات واحد فقط                 |
| `_طبقة_`        | `LAYER` | `LLAMA_SPLIT_MODE_LAYER`        | تقسيم الطبقات وذاكرة م-ق عبر المعالجات |
| `_صف_`          | `ROW` |   `LLAMA_SPLIT_MODE_ROW`          | تقسيم مع توازي الموترات إن دُعم         |

### نـوع_تحجيم_تمد / RopeScalingType

نوع تحجيم التموضع الدوراني (RoPE).

| القيمة بالعربية | القيمة بالإنجليزية                     | قيمة llama.cpp | الوصف          |
|-----------------|---------------------------------------|----------------|----------------|
| `_غير_محدد_`    | `UNSPECIFIED` | `LLAMA_ROPE_SCALING_TYPE_UNSPECIFIED`  | غير محدد       |
| `_لا_شيء_`       | `NONE` |        `LLAMA_ROPE_SCALING_TYPE_NONE`         | بدون تحجيم     |
| `_خطي_`         | `LINEAR` |      `LLAMA_ROPE_SCALING_TYPE_LINEAR`       | تحجيم خطي      |
| `_خيط_`         | `YARN` |        `LLAMA_ROPE_SCALING_TYPE_YARN`         | تحجيم YaRN     |
| `_تمد_طويل_`    | `LONGROPE` |    `LLAMA_ROPE_SCALING_TYPE_LONGROPE`     | تحجيم LongRoPE |

### نـوع_تجميع / PoolingType

نوع التجميع للتضمينات.

| القيمة بالعربية | القيمة بالإنجليزية                | قيمة llama.cpp | الوصف             |
|-----------------|----------------------------------|----------------|-------------------|
| `_غير_محدد_`    | `UNSPECIFIED` | `LLAMA_POOLING_TYPE_UNSPECIFIED`  | غير محدد          |
| `_لا_شيء_`       | `NONE` |        `LLAMA_POOLING_TYPE_NONE`         | بدون تجميع        |
| `_متوسط_`       | `MEAN` |        `LLAMA_POOLING_TYPE_MEAN`         | تجميع المتوسط     |
| `_فئة_`         | `CLS` |         `LLAMA_POOLING_TYPE_CLS`          | تجميع رمز CLS     |
| `_أخير_`        | `LAST` |        `LLAMA_POOLING_TYPE_LAST`         | تجميع الرمز الأخير |
| `_رتبة_`        | `RANK` |        `LLAMA_POOLING_TYPE_RANK`         | تجميع الرتبة      |

### نـوع_انتباه / AttentionType

نوع آلية الانتباه.

| القيمة بالعربية | القيمة بالإنجليزية                  | قيمة llama.cpp | الوصف                      |
|-----------------|------------------------------------|----------------|----------------------------|
| `_غير_محدد_`    | `UNSPECIFIED` | `LLAMA_ATTENTION_TYPE_UNSPECIFIED`  | غير محدد                   |
| `_سببي_`        | `CAUSAL` |      `LLAMA_ATTENTION_TYPE_CAUSAL`       | انتباه سببي (انحداري ذاتي) |
| `_غير_سببي_`    | `NON_CAUSAL` |  `LLAMA_ATTENTION_TYPE_NON_CAUSAL`   | انتباه غير سببي            |

### نـوع_انتباه_خاطف / FlashAttnType

إعدادات الانتباه الخاطف.

| القيمة بالعربية | القيمة بالإنجليزية | القيمة     | الوصف               |
|-----------------|-------------------|------------|---------------------|
| `_تلقائي_`      | `AUTO` |              `-1`     | اختيار تلقائي       |
| `_معطل_`        | `DISABLED` |               `0` | الانتباه الخاطف معطل |
| `_مفعل_`        | `ENABLED` |               `1`  | الانتباه الخاطف مفعل |

---

## هياكل البيانات

### بـيانات_رمز / TokenData

بيانات الرمز مع معلومات الاحتمالية. يقابل `llama_token_data`.

**الحقول:**

- `المعرف` / `id` (`رمـز` / `Token`) - معرف الرمز.
- `اللوجت` / `logit` (`عائم` / `Float`) - اللوغاريتم الاحتمالي.
- `الاحتمالية` / `p` (`عائم` / `Float`) - الاحتمالية.

### مـصفوفة_بيانات_رموز / TokenDataArray

مصفوفة بيانات الرموز للمعاينة. يقابل `llama_token_data_array`.

**الحقول:**

- `البيانات` / `data` (`مؤشر[بـيانات_رمز]` / `ptr[TokenData]`) - مؤشر لمصفوفة بيانات الرموز.
- `الحجم` / `size` (`طـبيعي_متكيف` / `ArchWord`) - عدد العناصر.
- `المختار` / `selected` (`صحيح[64]` / `Int[64]`) - فهرس الرمز المختار.
- `مرتب` / `sorted` (`ثـنائي` / `Bool`) - هل المصفوفة مرتبة.

### رسـالة_محادثة / ChatMessage

هيكل رسالة المحادثة لقوالب المحادثة. يقابل `llama_chat_message`.

**الحقول:**

- `الدور` / `role` (`مـؤشر_محارف` / `CharsPtr`) - دور الرسالة ("user"، "assistant"، "system").
- `المحتوى` / `content` (`مـؤشر_محارف` / `CharsPtr`) - محتوى الرسالة.

### بـيانات_سياق_أداء / PerfContextData

بيانات الأداء لعمليات السياق. يقابل `llama_perf_context_data`.

**الحقول:**

- `وقت_البدء_بالمللي` / `tStartMs` (`عائم[64]` / `Float[64]`) - وقت البدء بالمللي ثانية.
- `وقت_التحميل_بالمللي` / `tLoadMs` (`عائم[64]` / `Float[64]`) - وقت التحميل بالمللي ثانية.
- `وقت_التقييم_الأولي_بالمللي` / `tPEvalMs` (`عائم[64]` / `Float[64]`) - وقت تقييم المحث.
- `وقت_التقييم_بالمللي` / `tEvalMs` (`عائم[64]` / `Float[64]`) - وقت تقييم الرموز.
- `عدد_التقييم_الأولي` / `nPEval` (`صحيح` / `Int`) - عدد تقييمات المحث.
- `عدد_التقييم` / `nEval` (`صحيح` / `Int`) - عدد تقييمات الرموز.
- `عدد_المعاد_استخدامه` / `nReused` (`صحيح` / `Int`) - عدد التقييمات المُعاد استخدامها.

### بـيانات_معاين_أداء / PerfSamplerData

بيانات الأداء لعمليات المعاين. يقابل `llama_perf_sampler_data`.

**الحقول:**

- `وقت_المعاينة_بالمللي` / `tSampleMs` (`عائم[64]` / `Float[64]`) - وقت المعاينة بالمللي ثانية.
- `عدد_العينات` / `nSample` (`صحيح` / `Int`) - عدد العينات.

---

## الأصناف

### نـموذج / Model

يمثل نموذج لغة كبير محمّل. يقابل `llama_model`.

#### نـموذج.مـعطيات / Model.Params

معطيات تحميل النموذج. يقابل `llama_model_params`.

**الحقول:**

- `الأجهزة` / `devices` (`مؤشر` / `ptr`) - قائمة الأجهزة.
- `استدراكات_نوع_صوان_الموتر` / `tensorBuftOverrides` (`سند[مصفوفة[اسـتدراك_نوع_صوان_موتر]]` / `ref[array[TensorBuftOverride]]`) - استدراكات نوع صوان الموتر.
- `عدد_طبقات_المعالج_الرسومي` / `nGpuLayers` (`صحيح` / `Int`) - عدد الطبقات المُفرَّغة لمعالج الرسوميات.
- `نمط_التقسيم` / `splitMode` (`نـمط_تقسيم` / `SplitMode`) - نمط تقسيم معالج الرسوميات.
- `المعالج_الرسومي_الرئيسي` / `mainGpu` (`صحيح` / `Int`) - فهرس معالج الرسوميات الرئيسي.
- `تقسيم_الموتر` / `tensorSplit` (`سند[مصفوفة[عائم]]` / `ref[array[Float]]`) - نسب تقسيم الموتر.
- `دالة_التقدم` / `progressCallback` (`دالـة_تقدم` / `ProgressCallback`) - دالة التقدم.
- `بيانات_دالة_التقدم` / `progressCallbackUserData` (`مؤشر` / `ptr`) - بيانات المستخدم للدالة.
- `تجاوزات_مق` / `kvOverrides` (`مؤشر` / `ptr`) - تجاوزات ذاكرة م-ق (مفتاح - قيمة).
- `مفردات_فقط` / `vocabOnly` (`ثـنائي` / `Bool`) - تحميل المفردات فقط.
- `استخدم_خريطة_ذاكرة` / `useMmap` (`ثـنائي` / `Bool`) - استخدام خريطة الذاكرة.
- `استخدم_دخلا_مباشرا` / `useDirectIo` (`ثـنائي` / `Bool`) - استخدام الإدخال/الإخراج المباشر.
- `استخدم_قفل_ذاكرة` / `useMlock` (`ثـنائي` / `Bool`) - قفل الذاكرة.
- `تفحص_الموترات` / `checkTensors` (`ثـنائي` / `Bool`) - التحقق من بيانات الموترات.
- `استخدم_أنواع_صوان_إضافية` / `useExtraBufts` (`ثـنائي` / `Bool`) - استخدام أنواع صوان إضافية.
- `بلا_مضيف` / `noHost` (`ثـنائي` / `Bool`) - بدون تخصيص صوان المضيف.
- `بلا_حجز` / `noAlloc` (`ثـنائي` / `Bool`) - بدون تخصيص ذاكرة.

#### الدالات

- `نـموذج.هات_معطيات_افتراضية` / `Model.getDefaultParams`

`نـموذج.هات_معطيات_افتراضية(): مـعطيات`

<div dir=ltr>

`Model.getDefaultParams(): Params`

</div>

الحصول على معطيات النموذج الافتراضية. يقابل `llama_model_default_params`.

- `نـموذج.حمل` / `Model.load`

`نـموذج.حمل(مسار: مـؤشر_محارف، معطيات: مـعطيات): سند[نـموذج]`

<div dir=ltr>

`Model.load(path: CharsPtr, params: Params): ref[Model]`

</div>

تحميل النموذج من ملف. يقابل `llama_model_load_from_file`.

- `نـموذج.حمل_أجزاء` / `Model.loadSplits`

`نـموذج.حمل_أجزاء(مسارات: سند[مصفوفة[مـؤشر_محارف]]، عدد: كلمة، معطيات: مـعطيات): سند[نـموذج]`

<div dir=ltr>

`Model.loadSplits(paths: ref[array[CharsPtr]], count: Word, params: Params): ref[Model]`

</div>

تحميل النموذج من ملفات مجزأة. يقابل `llama_model_load_from_splits`.

- `نـموذج.حرر` / `Model.free`

`نـموذج.حرر(نموذج: سند[نـموذج])`

<div dir=ltr>

`Model.free(model: ref[Model])`

</div>

تحرير موارد النموذج. يقابل `llama_model_free`.

#### الوظائف والخصال

- `نموذج.احفظ` / `model.save`

`نموذج.احفظ(مسار: مـؤشر_محارف)`

<div dir=ltr>

`model.save(path: CharsPtr)`

</div>

حفظ النموذج في ملف. يقابل `llama_model_save_to_file`.

- `نموذج.لديه_مرمز: ثـنائي` / `model.hasEncoder: Bool`
  التحقق من وجود مرمز. يقابل `llama_model_has_encoder`.

- `نموذج.لديه_فاك_ترميز: ثـنائي` / `model.hasDencoder: Bool`
  التحقق من وجود فاك ترميز. يقابل `llama_model_has_dencoder`.

- `نموذج.عدد_سياقات_التدريب: صـحيح` / `model.nCtxTrain: Int`
  حجم سياق التدريب. يقابل `llama_model_n_ctx_train`.

- `نموذج.عدد_التضمينات: صـحيح` / `model.nEmbd: Int`
  بُعد التضمين. يقابل `llama_model_n_embd`.

- `نموذج.عدد_الطبقات: صـحيح` / `model.nLayer: Int`
  عدد الطبقات. يقابل `llama_model_n_layer`.

- `نموذج.عدد_الرؤوس: صـحيح` / `model.nHead: Int`
  عدد رؤوس الانتباه. يقابل `llama_model_n_head`.

- `نموذج.المفردات: سند[مـفردات]` / `model.vocab: ref[Vocab]`
  الحصول على المفردات. يقابل `llama_model_get_vocab`.

- نموذج.هات_بيانات_وصفية_كنص / `model.metaValStr`

```
نموذج.هات_بيانات_وصفية_كنص(
    مفتاح: مـؤشر_محارف، صوان: مـؤشر_محارف، حجم_الصوان: طـبيعي_متكيف
): صـحيح
```

<div dir=ltr>

`model.metaValStr(key: CharsPtr, buf: CharsPtr, bufSize: ArchWord): Int`

</div>

الحصول على قيمة بيانات وصفية كنص. يقابل `llama_model_meta_val_str`.

- `نموذج.عدد_البيانات_الوصفية: صـحيح` / `model.metaCount: Int`
  الحصول على عدد البيانات الوصفية. يقابل `llama_model_meta_count`.

- `نموذج.مفتاح_بيانات_وصفية_بفهرس` / `model.metaKeyByIndex`

```
نموذج.مفتاح_بيانات_وصفية_بفهرس(
    فهرس: صـحيح، صوان: مـؤشر_محارف، حجم_الصوان: طـبيعي_متكيف
): صـحيح
```

<div dir=ltr>

`model.metaKeyByIndex(idx: Int, buf: CharsPtr, bufSize: ArchWord): Int`

</div>

الحصول على مفتاح بيانات وصفية بالفهرس. يقابل `llama_model_meta_key_by_index`.

---

### سـياق / Context

سياق الاستدلال للنموذج. يقابل `llama_context`.

#### سـياق.مـعطيات / Context.Params

معطيات إنشاء السياق. يقابل `llama_context_params`.

**الحقول:**

- `رقم_السياق` / `nCtx` (`كلمة` / `Word`) - حجم السياق (0 = استخدام الافتراضي). يقابل `n_ctx`.
- `حجم_الدفعة_الظاهري` / `nBatch` (`كلمة` / `Word`) - حجم الدفعة المنطقي. يقابل `n_batch`.
- `حجم_الدفعة_الحقيقي` / `nUbatch` (`كلمة` / `Word`) - حجم الدفعة الفعلي. يقابل `n_ubatch`.
- `أقصى_تسلسل` / `nSeqMax` (`كلمة` / `Word`) - أقصى عدد تسلسلات. يقابل `n_seq_max`.
- `عدد_المسالك` / `nThreads` (`صحيح` / `Int`) - عدد المسالك للتوليد. يقابل `n_threads`.
- `عدد_مسالك_الدفعة` / `nThreadsBatch` (`صحيح` / `Int`) - عدد مسالك معالجة الدفعة. يقابل `n_threads_batch`.
- `نوع_تحجيم_تمد` / `ropeScalingType` (`نـوع_تحجيم_تمد` / `RopeScalingType`) - نوع تحجيم RoPE. يقابل `rope_scaling_type`.
- `نوع_التجميع` / `poolingType` (`نـوع_تجميع` / `PoolingType`) - نوع التجميع. يقابل `pooling_type`.
- `نوع_الانتباه` / `attentionType` (`نـوع_انتباه` / `AttentionType`) - نوع الانتباه. يقابل `attention_type`.
- `نوع_الانتباه_الخاطف` / `flashAttnType` (`نـوع_انتباه_خاطف` / `FlashAttnType`) - إعداد الانتباه الخاطف. يقابل `flash_attn`.
- `تردد_تمد_الأساسي` / `ropeFreqBase` (`عائم` / `Float`) - تردد RoPE الأساسي. يقابل `rope_freq_base`.
- `مقياس_تردد_تمد` / `ropeFreqScale` (`عائم` / `Float`) - مقياس تردد RoPE. يقابل `rope_freq_scale`.
- `عامل_استقراء_الخيط` / `yarnExtFactor` (`عائم` / `Float`) - عامل استقراء YaRN. يقابل `yarn_ext_factor`.
- `عامل_انتباه_الخيط` / `yarnAttnFactor` (`عائم` / `Float`) - عامل انتباه YaRN. يقابل `yarn_attn_factor`.
- `بيتا_خيط_سريع` / `yarnBetaFast` (`عائم` / `Float`) - بيتا YaRN السريع. يقابل `yarn_beta_fast`.
- `بيتا_خيط_بطيء` / `yarnBetaSlow` (`عائم` / `Float`) - بيتا YaRN البطيء. يقابل `yarn_beta_slow`.
- `سياق_خيط_أصلي` / `yarnOrigCtx` (`كلمة` / `Word`) - حجم سياق YaRN الأصلي. يقابل `yarn_orig_ctx`.
- `عتبة_الرص` / `defragThold` (`عائم` / `Float`) - عتبة إزالة التجزئة. يقابل `defrag_thold`.
- `دالة_التقييم` / `cbEval` (`مؤشر دالة` / `function ptr`) - دالة التقييم. يقابل `cb_eval`.
- `بيانات_دالة_التقييم` / `cbEvalData` (`مؤشر` / `ptr`) - بيانات مستخدم دالة التقييم. يقابل `cb_eval_user_data`.
- `نوع_م` / `typeK` (`Ggml.Type`) - نوع بيانات ذاكرة م. يقابل `type_k`.
- `نوع_ق` / `typeV` (`Ggml.Type`) - نوع بيانات ذاكرة ق. يقابل `type_v`.
- `دالة_الإلغاء` / `abortCb` (`مؤشر دالة` / `function ptr`) - دالة الإلغاء. يقابل `abort_callback`.
- `بيانات_دالة_الإلغاء` / `abortCbData` (`مؤشر` / `ptr`) - بيانات دالة الإلغاء. يقابل `abort_callback_data`.
- `التضمينات` / `embeddings` (`ثـنائي` / `Bool`) - تفعيل إخراج التضمينات. يقابل `embeddings`.
- `تفريغ_متق` / `offloadKqv` (`ثـنائي` / `Bool`) - تفريغ م-ت-ق لمعالج الرسوميات. يقابل `offload_kqv`.
- `بلا_أداء` / `noPerf` (`ثـنائي` / `Bool`) - تعطيل عدادات الأداء. يقابل `no_perf`.
- `تفريغ_عمليات` / `opOffload` (`ثـنائي` / `Bool`) - تفعيل تفريغ العمليات. يقابل `op_offload`.
- `ذنم_كاملة_الحجم` / `swaFull` (`ثـنائي` / `Bool`) - انتباه النافذة المنزلقة الكامل. يقابل `swa_full`.
- `مق_موحد` / `kvUnified` (`ثـنائي` / `Bool`) - ذاكرة م-ق موحدة. يقابل `kv_unified`.
- `المعاينات` / `samplers` (`سند[مصفوفة[إعـدادات_تسلسل_معاين]]` / `ref[array[SamplerSeqConfig]]`) - معاينات لكل تسلسل. يقابل `samplers`.
- `عدد_المعاينات` / `nSamplers` (`طـبيعي_متكيف` / `ArchWord`) - عدد المعاينات. يقابل `n_samplers`.

#### الدالات

- `سـياق.هات_معطيات_افتراضية` / `Context.getDefaultParams`

`سـياق.هات_معطيات_افتراضية(): مـعطيات`

<div dir=ltr>

`Context.getDefaultParams(): Params`

</div>

الحصول على معطيات السياق الافتراضية. يقابل `llama_context_default_params`.

- `سـياق.هيئ_من_نموذج` / `Context.initFromModel`

`سـياق.هيئ_من_نموذج(نموذج: سند[نـموذج]، معطيات: مـعطيات): سند[سـياق]`

<div dir=ltr>

`Context.initFromModel(model: ref[Model], params: Params): ref[Context]`

</div>

إنشاء سياق من نموذج. يقابل `llama_init_from_model`.

- `سـياق.حرر` / `Context.free`

`سـياق.حرر(سياق: سند[سـياق])`

<div dir=ltr>

`Context.free(ctx: ref[Context])`

</div>

تحرير موارد السياق. يقابل `llama_free`.

#### الوظائف والخصال

- `سياق.أرفق_تجميع_مسالك` / `ctx.attachThreadpool`

`سياق.أرفق_تجميع_مسالك(تج: سند[Ggml.Threadpool]، تج_دفعة: سند[Ggml.Threadpool])`

<div dir=ltr>

`ctx.attachThreadpool(tp: ref[Ggml.Threadpool], tpBatch: ref[Ggml.Threadpool])`

</div>

إرفاق تجميع مسالك. يقابل `llama_attach_threadpool`.

- `سياق.افصل_تجميع_مسالك` / `ctx.detachThreadpool`

`سياق.افصل_تجميع_مسالك()`

<div dir=ltr>

`ctx.detachThreadpool()`

</div>

فصل تجميع المسالك. يقابل `llama_detach_threadpool`.

- `سياق.هات_تسلسل_تضمينات` / `ctx.getEmbeddingsSeq`

`سياق.هات_تسلسل_تضمينات(معرف_التسلسل: مـعرف_تسلسل): سند[مصفوفة[عائم]]`

<div dir=ltr>

`ctx.getEmbeddingsSeq(seqId: SeqId): ref[array[Float]]`

</div>

الحصول على تضمينات التسلسل. يقابل `llama_get_embeddings_seq`.

- `سياق.هات_النموذج` / `ctx.getModel`

`سياق.هات_النموذج(): سند[نـموذج]`

<div dir=ltr>

`ctx.getModel(): ref[Model]`

</div>

الحصول على النموذج المرتبط. يقابل `llama_get_model`.

- `سياق.رقم_السياق: كلمة` / `ctx.nCtx: Word`
  الحصول على حجم السياق. يقابل `llama_n_ctx`.

- `سياق.حجم_الدفعة_الظاهري: كلمة` / `ctx.nBatch: Word`
  الحصول على حجم الدفعة المنطقي. يقابل `llama_n_batch`.

- `سياق.حجم_الدفعة_الحقيقي: كلمة` / `ctx.nUbatch: Word`
  الحصول على حجم الدفعة الفعلي. يقابل `llama_n_ubatch`.

- `سياق.أقصى_تسلسل: كلمة` / `ctx.nSeqMax: Word`
  الحصول على أقصى تسلسلات. يقابل `llama_n_seq_max`.

- `سياق.هات_الذاكرة` / `ctx.getMemory`

`سياق.هات_الذاكرة(): مؤشر[ذاكـرة]`

<div dir=ltr>

`ctx.getMemory(): ptr[Memory]`

</div>

الحصول على ذاكرة م-ق. يقابل `llama_get_memory`.

- `سياق.نوع_التجميع: نـوع_تجميع` / `ctx.poolingType: PoolingType`
  الحصول على نوع التجميع. يقابل `llama_pooling_type`.

- `سياق.رمز` / `ctx.encode`

`سياق.رمز(دفعة: دفـعة): صـحيح`

<div dir=ltr>

`ctx.encode(batch: Batch): Int`

</div>

ترميز الدفعة (لنماذج المرمز). يقابل `llama_encode`.

- `سياق.فك_ترميز` / `ctx.decode`

`سياق.فك_ترميز(دفعة: دفـعة): صـحيح`

<div dir=ltr>

`ctx.decode(batch: Batch): Int`

</div>

فك ترميز الدفعة. يقابل `llama_decode`.

- `سياق.هات_لوجتات_ث` / `ctx.getLogitsIth`

`سياق.هات_لوجتات_ث(ث: صـحيح): سند[مصفوفة[عائم]]`

<div dir=ltr>

`ctx.getLogitsIth(i: Int): ref[array[Float]]`

</div>

الحصول على اللوجتات للرمز بالفهرس. يقابل `llama_get_logits_ith`.

- `سياق.هات_تضمينات_ث` / `ctx.getEmbeddingsIth`

`سياق.هات_تضمينات_ث(ث: صـحيح): سند[مصفوفة[عائم]]`

<div dir=ltr>

`ctx.getEmbeddingsIth(i: Int): ref[array[Float]]`

</div>

الحصول على التضمينات للرمز بالفهرس. يقابل `llama_get_embeddings_ith`.

- `سياق.حجم_الحالة: طـبيعي_متكيف` / `ctx.stateSize: ArchWord`
  الحصول على حجم الحالة بالبايت. يقابل `llama_state_get_size`.

- `سياق.هات_بيانات_الحالة` / `ctx.stateGetData`

`سياق.هات_بيانات_الحالة(وجهة: سند[مصفوفة[كلمة[8]]]، حجم: طـبيعي_متكيف): طـبيعي_متكيف`

<div dir=ltr>

`ctx.stateGetData(dst: ref[array[Word[8]]], size: ArchWord): ArchWord`

</div>

الحصول على بيانات الحالة. يقابل `llama_state_get_data`.

- `سياق.حدد_بيانات_الحالة` / `ctx.stateSetData`

`سياق.حدد_بيانات_الحالة(مصدر: سند[مصفوفة[كلمة[8]]]، حجم: طـبيعي_متكيف): طـبيعي_متكيف`

<div dir=ltr>

`ctx.stateSetData(src: ref[array[Word[8]]], size: ArchWord): ArchWord`

</div>

تحديد بيانات الحالة. يقابل `llama_state_set_data`.

- `سياق.حمل_ملف_حالة` / `ctx.stateLoadFile`

```
سياق.حمل_ملف_حالة(
    مسار: مـؤشر_محارف، رموز_الخرج: سند[مصفوفة[رمـز]]، سعة: طـبيعي_متكيف، عدد_الخرج: سند[طـبيعي_متكيف]
): ثـنائي
```

<div dir=ltr>

`ctx.stateLoadFile(path: CharsPtr, tokensOut: ref[array[Token]], cap: ArchWord, outCount: ref[ArchWord]): Bool`

</div>

تحميل الحالة من ملف. يقابل `llama_state_load_file`.

- `سياق.احفظ_ملف_حالة` / `ctx.stateSaveFile`

`سياق.احفظ_ملف_حالة(مسار: مـؤشر_محارف، رموز: سند[مصفوفة[رمـز]]، عدد: طـبيعي_متكيف): ثـنائي`

<div dir=ltr>

`ctx.stateSaveFile(path: CharsPtr, tokens: ref[array[Token]], count: ArchWord): Bool`

</div>

حفظ الحالة في ملف. يقابل `llama_state_save_file`.

- `سياق.حدد_موائم_تمر` / `ctx.setAdapterLora`

`سياق.حدد_موائم_تمر(موائم: سند[مـوائم]، مقياس: عائم): صـحيح`

<div dir=ltr>

`ctx.setAdapterLora(ad: ref[Adapter], scale: Float): Int`

</div>

تطبيق موائم LoRA. يقابل `llama_set_adapter_lora`.

- `سياق.أزل_موائم_تمر` / `ctx.removeAdapterLora`

`سياق.أزل_موائم_تمر(موائم: سند[مـوائم]): صـحيح`

<div dir=ltr>

`ctx.removeAdapterLora(ad: ref[Adapter]): Int`

</div>

إزالة موائم LoRA. يقابل `llama_rm_adapter_lora`.

- `سياق.صفر_موائم_تمر` / `ctx.clearAdapterLora`

`سياق.صفر_موائم_تمر()`

<div dir=ltr>

`ctx.clearAdapterLora()`

</div>

مسح جميع موائمات LoRA. يقابل `llama_clear_adapter_lora`.

- `سياق.سياق_الأداء: بـيانات_سياق_أداء` / `ctx.perfContext: PerfContextData`
  الحصول على بيانات الأداء. يقابل `llama_perf_context`.

- `سياق.اطبع_سياق_الأداء` / `ctx.perfContextPrint`

`سياق.اطبع_سياق_الأداء()`

<div dir=ltr>

`ctx.perfContextPrint()`

</div>

طباعة بيانات الأداء. يقابل `llama_perf_context_print`.

- `سياق.صفر_سياق_الأداء` / `ctx.perfContextReset`

`سياق.صفر_سياق_الأداء()`

<div dir=ltr>

`ctx.perfContextReset()`

</div>

إعادة تعيين عدادات الأداء. يقابل `llama_perf_context_reset`.

---

### دفـعة / Batch

دفعة رموز للمعالجة. يقابل `llama_batch`.

**الحقول:**

- `عدد_الرموز` / `nTokens` (`صحيح` / `Int`) - عدد الرموز. يقابل `n_tokens`.
- `الرموز` / `token` (`سند[مصفوفة[رمـز]]` / `ref[array[Token]]`) - معرفات الرموز. يقابل `token`.
- `التضمين` / `embd` (`سند[مصفوفة[عائم]]` / `ref[array[Float]]`) - التضمينات (بديل للرموز). يقابل `embd`.
- `الموقع` / `pos` (`سند[مـوقع]` / `ref[Pos]`) - مواقع الرموز. يقابل `pos`.
- `عدد_معرفات_التسلسل` / `nSeqId` (`سند[صحيح]` / `ref[Int]`) - عدد معرفات التسلسل لكل رمز. يقابل `n_seq_id`.
- `معرفات_التسلسل` / `seqId` (`سند[سند[مـعرف_تسلسل]]` / `ref[ref[SeqId]]`) - معرفات التسلسل. يقابل `seq_id`.
- `الخرج` / `output` (`سند[صحيح[8]]` / `ref[Int[8]]`) - أعلام الإخراج. يقابل `logits`.

#### الدالات

- `دفـعة.هات_واحدا` / `Batch.getOne`

`دفـعة.هات_واحدا(رموز: سند[مصفوفة[رمـز]]، عدد_الرموز: صـحيح): دفـعة`

<div dir=ltr>

`Batch.getOne(tokens: ref[array[Token]], nTokens: Int): Batch`

</div>

إنشاء دفعة من مصفوفة رموز. يقابل `llama_batch_get_one`.

- `دفـعة.هيئ` / `Batch.init`

`دفـعة.هيئ(عدد_الرموز: صـحيح، تضمين: صـحيح، أقصى_تسلسل: صـحيح): دفـعة`

<div dir=ltr>

`Batch.init(nTokens: Int, embd: Int, nSeqMax: Int): Batch`

</div>

تهيئة دفعة فارغة. يقابل `llama_batch_init`.

---

### مـعاين / Sampler

معاين رموز للتوليد. يقابل `llama_sampler`.

#### مـعاين.مـعطيات_سلسلة / Sampler.ChainParams

معطيات سلسلة المعاين. يقابل `llama_sampler_chain_params`.

**الحقول:**

- `بلا_أداء` / `noPerf` (`ثـنائي` / `Bool`) - تعطيل عدادات الأداء. يقابل `no_perf`.

#### الدالات

- `مـعاين.هيئ` / `Sampler.init`

`مـعاين.هيئ(واجهة: مؤشر، سياق: مؤشر): سند[مـعاين]`

<div dir=ltr>

`Sampler.init(iface: ptr, ctx: ptr): ref[Sampler]`

</div>

تهيئة معاين مخصص. يقابل `llama_sampler_init`.

- `مـعاين.هيئ_سلسلة` / `Sampler.chainInit`

`مـعاين.هيئ_سلسلة(معطيات: مـعطيات_سلسلة): سند[مـعاين]`

<div dir=ltr>

`Sampler.chainInit(params: ChainParams): ref[Sampler]`

</div>

إنشاء سلسلة معاينات. يقابل `llama_sampler_chain_init`.

- `مـعاين.هيئ_جشع` / `Sampler.initGreedy`

`مـعاين.هيئ_جشع(): سند[مـعاين]`

<div dir=ltr>

`Sampler.initGreedy(): ref[Sampler]`

</div>

إنشاء معاين جشع. يقابل `llama_sampler_init_greedy`.

- `مـعاين.هيئ_منتشر` / `Sampler.initDist`

`مـعاين.هيئ_منتشر(بذرة: كلمة): سند[مـعاين]`

<div dir=ltr>

`Sampler.initDist(seed: Word): ref[Sampler]`

</div>

إنشاء معاين توزيع. يقابل `llama_sampler_init_dist`.

- `مـعاين.هيئ_أعلى_ع` / `Sampler.initTopK`

`مـعاين.هيئ_أعلى_ع(ع: صـحيح): سند[مـعاين]`

<div dir=ltr>

`Sampler.initTopK(k: Int): ref[Sampler]`

</div>

إنشاء معاين أعلى-ع. يقابل `llama_sampler_init_top_k`.

- `مـعاين.هيئ_أعلى_ن` / `Sampler.initTopP`

`مـعاين.هيئ_أعلى_ن(ن: عائم، أدنى_إبقاء: طـبيعي_متكيف): سند[مـعاين]`

<div dir=ltr>

`Sampler.initTopP(p: Float, minKeep: ArchWord): ref[Sampler]`

</div>

إنشاء معاين أعلى-ن (النواة). يقابل `llama_sampler_init_top_p`.

- `مـعاين.هيئ_جزاءات` / `Sampler.initPenalties`

```
مـعاين.هيئ_جزاءات(
    جزاء_آخر_ن: صـحيح، جزاء_التكرار: عائم، جزاء_التردد: عائم، جزاء_الوجود: عائم
): سند[مـعاين]
```

<div dir=ltr>

`Sampler.initPenalties(penaltyLastN: Int, penaltyRepeat: Float, penaltyFreq: Float, penaltyPresent: Float): ref[Sampler]`

</div>

إنشاء معاين الجزاءات. يقابل `llama_sampler_init_penalties`.
- `جزاء_آخر_ن` / `penaltyLastN`: آخر ن رمز للمعاقبة (0 = تعطيل، -1 = حجم السياق)
- `جزاء_التكرار` / `penaltyRepeat`: جزاء التكرار (1.0 = معطل)
- `جزاء_التردد` / `penaltyFreq`: جزاء التردد (0.0 = معطل)
- `جزاء_الوجود` / `penaltyPresent`: جزاء الوجود (0.0 = معطل)

- `مـعاين.انسخ` / `Sampler.clone`

`مـعاين.انسخ(م: سند[مـعاين]): سند[مـعاين]`

<div dir=ltr>

`Sampler.clone(s: ref[Sampler]): ref[Sampler]`

</div>

نسخ المعاين. يقابل `llama_sampler_clone`.

- `مـعاين.حرر` / `Sampler.free`

`مـعاين.حرر(م: سند[مـعاين])`

<div dir=ltr>

`Sampler.free(s: ref[Sampler])`

</div>

تحرير موارد المعاين. يقابل `llama_sampler_free`.

#### الوظائف والخصال

- `معاين.صفر` / `sampler.reset`

`معاين.صفر()`

<div dir=ltr>

`sampler.reset()`

</div>

إعادة تعيين حالة المعاين. يقابل `llama_sampler_reset`.

- `معاين.عاين` / `sampler.sample`

`معاين.عاين(سياق: سند[سـياق]، فهرس: صـحيح): رمـز`

<div dir=ltr>

`sampler.sample(ctx: ref[Context], idx: Int): Token`

</div>

معاينة الرمز التالي. يقابل `llama_sampler_sample`.

- `معاين.الاسم: مـؤشر_محارف` / `sampler.name: CharsPtr`
  الحصول على اسم المعاين. يقابل `llama_sampler_name`.

- `معاين.اقبل` / `sampler.accept`

`معاين.اقبل(رمز: رمـز)`

<div dir=ltr>

`sampler.accept(token: Token)`

</div>

قبول الرمز للتتبع. يقابل `llama_sampler_accept`.

- `معاين.طبق` / `sampler.apply`

`معاين.طبق(مرشحون: سند[مـصفوفة_بيانات_رموز])`

<div dir=ltr>

`sampler.apply(cand: ref[TokenDataArray])`

</div>

تطبيق المعاين على المرشحين. يقابل `llama_sampler_apply`.

- `معاين.أضف_للسلسلة` / `sampler.chainAdd`

`معاين.أضف_للسلسلة(م: سند[مـعاين])`

<div dir=ltr>

`sampler.chainAdd(s: ref[Sampler])`

</div>

إضافة معاين للسلسلة. يقابل `llama_sampler_chain_add`.

- `معاين.هات_من_السلسلة` / `sampler.chainGet`

`معاين.هات_من_السلسلة(فهرس: صـحيح): سند[مـعاين]`

<div dir=ltr>

`sampler.chainGet(idx: Int): ref[Sampler]`

</div>

الحصول على معاين من السلسلة. يقابل `llama_sampler_chain_get`.

- `معاين.عدد_السلسلة: صـحيح` / `sampler.chainCount: Int`
  الحصول على طول السلسلة. يقابل `llama_sampler_chain_n`.

- `معاين.أزل_من_السلسلة` / `sampler.chainRemove`

`معاين.أزل_من_السلسلة(فهرس: صـحيح): سند[مـعاين]`

<div dir=ltr>

`sampler.chainRemove(idx: Int): ref[Sampler]`

</div>

إزالة معاين من السلسلة. يقابل `llama_sampler_chain_remove`.

- `معاين.معاين_الأداء: بـيانات_معاين_أداء` / `sampler.perfSampler: PerfSamplerData`
  الحصول على بيانات الأداء. يقابل `llama_perf_sampler`.

- `معاين.اطبع_معاين_الأداء` / `sampler.perfSamplerPrint`

`معاين.اطبع_معاين_الأداء()`

<div dir=ltr>

`sampler.perfSamplerPrint()`

</div>

طباعة بيانات الأداء. يقابل `llama_perf_sampler_print`.

- `معاين.صفر_معاين_الأداء` / `sampler.perfSamplerReset`

`معاين.صفر_معاين_الأداء()`

<div dir=ltr>

`sampler.perfSamplerReset()`

</div>

إعادة تعيين عدادات الأداء. يقابل `llama_perf_sampler_reset`.

---

### مـفردات / Vocab

مفردات النموذج. يقابل `llama_vocab`.

#### الوظائف والخصال

- `مفردات.رمز_إلى_قطعة` / `vocab.tokenToPiece`

`مفردات.رمز_إلى_قطعة(رمز: رمـز، صوان: مـؤشر_محارف، طول: صـحيح، قص_يسار: صـحيح، خاص: ثـنائي): صـحيح`

<div dir=ltr>

`vocab.tokenToPiece(token: Token, buf: CharsPtr, length: Int, lstrip: Int, special: Bool): Int`

</div>

تحويل الرمز إلى نص. يقابل `llama_token_to_piece`.

- `مفردات.نهاية_الجملة: رمـز` / `vocab.eos: Token`
  الحصول على رمز نهاية التسلسل. يقابل `llama_vocab_eos`.

- `مفردات.أهي_نهاية_التوليد` / `vocab.isEog`

`مفردات.أهي_نهاية_التوليد(رمز: رمـز): ثـنائي`

<div dir=ltr>

`vocab.isEog(token: Token): Bool`

</div>

التحقق إن كان الرمز نهاية التوليد. يقابل `llama_vocab_is_eog`.

---

### ذاكـرة / Memory

إدارة ذاكرة م-ق. يقابل `llama_memory`.

#### الوظائف والخصال

- `ذاكرة.صفر` / `memory.clear`

`ذاكرة.صفر(بيانات: ثـنائي)`

<div dir=ltr>

`memory.clear(data: Bool)`

</div>

مسح ذاكرة م-ق. يقابل `llama_memory_clear`.

- `ذاكرة.أزل_تسلسلا` / `memory.seqRemove`

`ذاكرة.أزل_تسلسلا(تسلسل: مـعرف_تسلسل، م0: مـوقع، م1: مـوقع): ثـنائي`

<div dir=ltr>

`memory.seqRemove(seq: SeqId, p0: Pos, p1: Pos): Bool`

</div>

إزالة نطاق تسلسل. يقابل `llama_memory_seq_rm`.

- `ذاكرة.انسخ_تسلسلا` / `memory.seqCopy`

`ذاكرة.انسخ_تسلسلا(مصدر: مـعرف_تسلسل، وجهة: مـعرف_تسلسل، م0: مـوقع، م1: مـوقع)`

<div dir=ltr>

`memory.seqCopy(src: SeqId, dst: SeqId, p0: Pos, p1: Pos)`

</div>

نسخ نطاق تسلسل. يقابل `llama_memory_seq_cp`.

- `ذاكرة.أبق_تسلسلا` / `memory.seqKeep`

`ذاكرة.أبق_تسلسلا(تسلسل: مـعرف_تسلسل)`

<div dir=ltr>

`memory.seqKeep(seq: SeqId)`

</div>

الإبقاء على التسلسل المحدد فقط. يقابل `llama_memory_seq_keep`.

- `ذاكرة.أضف_لتسلسل` / `memory.seqAdd`

`ذاكرة.أضف_لتسلسل(تسلسل: مـعرف_تسلسل، م0: مـوقع، م1: مـوقع، فرق: مـوقع)`

<div dir=ltr>

`memory.seqAdd(seq: SeqId, p0: Pos, p1: Pos, delta: Pos)`

</div>

إضافة فرق موقع للتسلسل. يقابل `llama_memory_seq_add`.

- `ذاكرة.قسم_تسلسلا` / `memory.seqDiv`

`ذاكرة.قسم_تسلسلا(تسلسل: مـعرف_تسلسل، م0: مـوقع، م1: مـوقع، ق: صـحيح)`

<div dir=ltr>

`memory.seqDiv(seq: SeqId, p0: Pos, p1: Pos, d: Int)`

</div>

تقسيم المواقع في التسلسل. يقابل `llama_memory_seq_div`.

---

### مـوائم / Adapter

دعم موائم LoRA. يقابل `llama_adapter_lora`.

#### الدالات

- `مـوائم.تمر_هيئ` / `Adapter.loraInit`

`مـوائم.تمر_هيئ(نموذج: سند[نـموذج]، مسار: مـؤشر_محارف): سند[مـوائم]`

<div dir=ltr>

`Adapter.loraInit(model: ref[Model], path: CharsPtr): ref[Adapter]`

</div>

تحميل موائم LoRA. يقابل `llama_adapter_lora_init`.

- `مـوائم.تمر_حرر` / `Adapter.loraFree`

`مـوائم.تمر_حرر(موائم: سند[مـوائم])`

<div dir=ltr>

`Adapter.loraFree(ad: ref[Adapter])`

</div>

تحرير موائم LoRA. يقابل `llama_adapter_lora_free`.

---

## الدالات العمومية

### إدارة المشغل

- `هيئ_المشغل()`
  `backendInit()`
  تهيئة مشغل لاما. يقابل `llama_backend_init`.

- `حرر_المشغل()`
  `backendFree()`
  تحرير مشغل لاما. يقابل `llama_backend_free`.

- `هيئ_نوما(استراتيجية: Ggml.NumaStrategy)`
  `numaInit(strategy: Ggml.NumaStrategy)`
  تهيئة NUMA. يقابل `llama_numa_init`.

### معلومات النظام

- `الوقت_بالميكرو(): صـحيح[64]`
  `timeUs(): Int[64]`
  الحصول على الوقت الحالي بالميكروثانية. يقابل `llama_time_us`.

- `أقصى_أجهزة(): طـبيعي_متكيف`
  `maxDevices(): ArchWord`
  الحصول على أقصى عدد أجهزة. يقابل `llama_max_devices`.

- `أقصى_تسلسلات_متوازية(): طـبيعي_متكيف`
  `maxParallelSequences(): ArchWord`
  الحصول على أقصى تسلسلات متوازية. يقابل `llama_max_parallel_sequences`.

- `يدعم_خريطة_الذاكرة(): ثـنائي`
  `supportsMmap(): Bool`
  التحقق من دعم خريطة الذاكرة. يقابل `llama_supports_mmap`.

- `يدعم_قفل_الذاكرة(): ثـنائي`
  `supportsMlock(): Bool`
  التحقق من دعم قفل الذاكرة. يقابل `llama_supports_mlock`.

- `يدعم_تفويض_معالج_رسومي(): ثـنائي`
  `supportsGpuOffload(): Bool`
  التحقق من دعم التفويض لمعالج الرسوميات. يقابل `llama_supports_gpu_offload`.

- `يدعم_استدعاء_بعيدا(): ثـنائي`
  `supportsRpc(): Bool`
  التحقق من دعم RPC. يقابل `llama_supports_rpc`.

- `اطبع_معلومات_النظام(): مـؤشر_محارف`
  `printSystemInfo(): CharsPtr`
  الحصول على نص معلومات النظام. يقابل `llama_print_system_info`.

### الترميز

- `رمز` / `tokenize`

```
رمز(
    مفردات: سند[مـفردات]، نص: مـؤشر_محارف، طول_النص: صـحيح، رموز: سند[مصفوفة[رمـز]]،
    أقصى_رموز: صـحيح، أضف_خاصة: ثـنائي، حلل_خاصة: ثـنائي
): صـحيح
```

<div dir=ltr>

```
tokenize(
    vocab: ref[Vocab], text: CharsPtr, textLen: Int, tokens: ref[array[Token]],
    nTokensMax: Int, addSpecial: Bool, parseSpecial: Bool
): Int
```

</div>

تحويل النص إلى رموز. يُرجع عدد الرموز المكتوبة. يقابل `llama_tokenize`.

- `فك_ترميز` / `detokenize`

```
فك_ترميز(
    مفردات: سند[مـفردات]، رموز: سند[مصفوفة[رمـز]]، عدد_الرموز: صـحيح، نص: مـؤشر_محارف،
    أقصى_طول_نص: صـحيح، أزل_خاصة: ثـنائي، ألغ_تحليل_خاصة: ثـنائي
): صـحيح
```

<div dir=ltr>

```
detokenize(
    vocab: ref[Vocab], tokens: ref[array[Token]], nTokens: Int, text: CharsPtr,
    textLenMax: Int, removeSpecial: Bool, unparseSpecial: Bool
): Int
```

</div>

تحويل الرموز إلى نص. يُرجع عدد المحارف المكتوبة. يقابل `llama_detokenize`.

### قوالب المحادثة

- `طبق_قالب_محادثة` / `chatApplyTemplate`

```
طبق_قالب_محادثة(
    قالب: مـؤشر_محارف، محادثة: سند[مصفوفة[رسـالة_محادثة]]، عدد_الرسائل: طـبيعي_متكيف،
    أضف_مساعدا: ثـنائي، صوان: مـؤشر_محارف، طول_الصوان: صـحيح
): صـحيح
```

<div dir=ltr>

```
chatApplyTemplate(
    tmpl: CharsPtr, chat: ref[array[ChatMessage]], nMsg: ArchWord,
    addAssistant: Bool, buf: CharsPtr, bufLen: Int
): Int
```

</div>

تطبيق قالب المحادثة على الرسائل. مرر `0` لـ `قالب` / `tmpl` لاستخدام قالب النموذج الافتراضي. يُرجع عدد المحارف المكتوبة. يقابل `llama_chat_apply_template`.

- `قوالب_محادثة_مدمجة` / `chatBuiltinTemplates`

`قوالب_محادثة_مدمجة(خرج: سند[مـؤشر_محارف]، طول: طـبيعي_متكيف): صـحيح`

<div dir=ltr>

`chatBuiltinTemplates(out: ref[CharsPtr], len: ArchWord): Int`

</div>

الحصول على أسماء القوالب المدمجة. يقابل `llama_chat_builtin_templates`.

### التسجيل

- `حدد_دالة_تسجيل` / `logSet`

```
حدد_دالة_تسجيل(
    دالة: مؤشر[دالة (مستوى: صـحيح، نص: مـؤشر_محارف، بيانات_المستخدم: مؤشر)]، بيانات_المستخدم: مؤشر
)
```

<div dir=ltr>

`logSet(cb: ptr[function (level: Int, text: CharsPtr, userData: ptr)], userData: ptr)`

</div>

تحديد دالة التسجيل. يقابل `llama_log_set`.

---

## الأمثلة

### إكمال النص

انظر `Examples/إكمال.أسس` أو `Examples/completion.alusus` لمثال كامل على إكمال النص.

### المحادثة

انظر `Examples/محادثة.أسس` أو `Examples/chat.alusus` لمثال محادثة متعددة الأدوار مع قوالب المحادثة.

---

## الرخصة

تتبع هذه المكتبة نفس رخصة llama.cpp (رخصة MIT).

</div>

