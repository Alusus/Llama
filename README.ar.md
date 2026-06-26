# لـاما - روابط الأسس لمكتبة llama.cpp
[[English]](README.md)

<div dir=rtl>

روابط لغة الأسس لمكتبة [llama.cpp](https://github.com/ggerganov/llama.cpp)، توفر واجهة كاملة لتشغيل استدلال نماذج اللغة الكبيرة محلياً. تدعم هذه المكتبة المعالج المركزي ومعالج الرسوميات عبر Vulkan.

## التثبيت

```
اشمل "مـحا"؛
مـحا.اشمل_حزمة("Alusus/Llama@0.2"، "لـاما.أسس")؛
```

<div dir=ltr>

```
import "Apm";
Apm.importPackage("Alusus/Llama@0.2");
```

</div>

## مثال

```
اشمل "مـتم/طـرفية"؛
اشمل "مـحا"؛
مـحا.اشمل_حزمة("Alusus/Llama@0.2"، "لـاما.أسس")؛
استخدم مـتم؛
استخدم لـاما؛

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
Apm.importPackage("Alusus/Llama@0.2");
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

```
GGML_USE_VULKAN=1 alusus your_script.alusus
```

</div>

## مرجع الواجهة البرمجية

### الأنواع الأساسية

* `رمـز` / `Token`: `صـحيح` / `Int` - معرف الرمز (لقب لـ `llama_token` في llama.cpp)
* `مـوقع` / `Pos`: `صـحيح` / `Int` - موقع الرمز (لقب لـ `llama_pos` في llama.cpp)
* `مـعرف_تسلسل` / `SeqId`: `صـحيح` / `Int` - معرف التسلسل (لقب لـ `llama_seq_id` في llama.cpp)
* `دالـة_تقدم` / `ProgressCallback`: `مؤشر[دالة (تقدم: عـائم، بيانات_مستخدم: مؤشر): ثـنائي]` / `ptr[function (progress: Float, userData: ptr): Bool]` - مؤشر دالة التقدم (لقب لـ `llama_progress_callback` في llama.cpp)

### الثوابت

* `_البذرة_الافتراضية_` / `DEFAULT_SEED` (`LLAMA_DEFAULT_SEED`، `0xFFFFFFFF`): البذرة العشوائية الافتراضية
* `_رمز_عدمي_` / `TOKEN_NULL` (`LLAMA_TOKEN_NULL`، `-1`): قيمة الرمز العدمي

## السرود

### نـمط_تقسيم / SplitMode

يتحكم في كيفية توزيع طبقات النموذج على معالجات الرسوميات.

* `_لا_شيء_` / `NONE` (`LLAMA_SPLIT_MODE_NONE`): معالج رسوميات واحد فقط
* `_طبقة_` / `LAYER` (`LLAMA_SPLIT_MODE_LAYER`): تقسيم الطبقات وذاكرة م-ق عبر المعالجات
* `_صف_` / `ROW` (`LLAMA_SPLIT_MODE_ROW`): تقسيم مع توازي الموترات إن دُعم

### نـوع_تحجيم_تمد / RopeScalingType

نوع تحجيم التموضع الدوراني (RoPE).

* `_غير_محدد_` / `UNSPECIFIED`: غير محدد (تقابل `LLAMA_ROPE_SCALING_TYPE_UNSPECIFIED` في llama.cpp)
* `_لا_شيء_` / `NONE`: بدون تحجيم (تقابل `LLAMA_ROPE_SCALING_TYPE_NONE` في llama.cpp)
* `_خطي_` / `LINEAR`: تحجيم خطي (تقابل `LLAMA_ROPE_SCALING_TYPE_LINEAR` في llama.cpp)
* `_خيط_` / `YARN`: تحجيم YaRN (تقابل `LLAMA_ROPE_SCALING_TYPE_YARN` في llama.cpp)
* `_تمد_طويل_` / `LONGROPE`: تحجيم LongRoPE (تقابل `LLAMA_ROPE_SCALING_TYPE_LONGROPE` في llamma.cpp)

### نـوع_تجميع / PoolingType

نوع التجميع للتضمينات.

* `_غير_محدد_` / `UNSPECIFIED`: غير محدد (تقابل `LLAMA_POOLING_TYPE_UNSPECIFIED` في llama.cpp)
* `_لا_شيء_` / `NONE`: بدون تجميع (تقابل `LLAMA_POOLING_TYPE_NONE` في llama.cpp)
* `_متوسط_` / `MEAN`: تجميع المتوسط (تقابل `LLAMA_POOLING_TYPE_MEAN` في llama.cpp)
* `_فئة_` / `CLS`: تجميع رمز CLS (تقابل `LLAMA_POOLING_TYPE_CLS` في llama.cpp)
* `_أخير_` / `LAST`: تجميع الرمز الأخير (تقابل `LLAMA_POOLING_TYPE_LAST` في llama.cpp)
* `_رتبة_` / `RANK`: تجميع الرتبة (تقابل `LLAMA_POOLING_TYPE_RANK` في llama.cpp)

### نـوع_انتباه / AttentionType

نوع آلية الانتباه.

* `_غير_محدد_` / `UNSPECIFIED`: غير محدد (تقابل `LLAMA_ATTENTION_TYPE_UNSPECIFIED` في llama.cpp)
* `_سببي_` / `CAUSAL`: انتباه سببي (انحداري ذاتي) (تقابل `LLAMA_ATTENTION_TYPE_CAUSAL` في llama.cpp)
* `_غير_سببي_` / `NON_CAUSAL`: انتباه غير سببي (تقابل `LLAMA_ATTENTION_TYPE_NON_CAUSAL` في llama.cpp)

### نـوع_انتباه_خاطف / FlashAttnType

إعدادات الانتباه الخاطف.

* `_تلقائي_` / `AUTO`: اختيار تلقائي (قيمته -1)
* `_معطل_` / `DISABLED`: الانتباه الخاطف معطل (قيمته 0)
* `_مفعل_` / `ENABLED`: الانتباه الخاطف مفعل (قيمته 1)

## هياكل البيانات

### بـيانات_رمز / TokenData

بيانات الرمز مع معلومات الاحتمالية. يقابل `llama_token_data`.

* `المعرف` / `id` (`رمـز` / `Token`): معرف الرمز. يقابل `id`.
* `اللوجت` / `logit` (`عـائم` / `Float`): اللوغاريتم الاحتمالي. يقابل `logit`.
* `الاحتمالية` / `p` (`عـائم` / `Float`): الاحتمالية. يقابل `p`.

### مـصفوفة_بيانات_رموز / TokenDataArray

مصفوفة بيانات الرموز للمعاينة. يقابل `llama_token_data_array`.

* `البيانات` / `data` (`مؤشر[بـيانات_رمز]` / `ptr[TokenData]`): مؤشر لمصفوفة بيانات الرموز. يقابل `data`.
* `الحجم` / `size` (`طـبيعي_متكيف` / `ArchWord`): عدد العناصر. يقابل `size`.
* `المختار` / `selected` (`صـحيح[64]` / `Int[64]`): فهرس الرمز المختار. يقابل `selected`.
* `مرتب` / `sorted` (`ثـنائي` / `Bool`): هل المصفوفة مرتبة. يقابل `sorted`.

### رسـالة_محادثة / ChatMessage

هيكل رسالة المحادثة لقوالب المحادثة. يقابل `llama_chat_message`.

* `الدور` / `role` (`مـؤشر_محارف` / `CharsPtr`): دور الرسالة ("user"، "assistant"، "system"). يقابل `role`.
* `المحتوى` / `content` (`مـؤشر_محارف` / `CharsPtr`): محتوى الرسالة. يقابل `content`.

### بـيانات_سياق_أداء / PerfContextData

بيانات الأداء لعمليات السياق. يقابل `llama_perf_context_data`.

* `وقت_البدء_بالمللي` / `tStartMs` (`عـائم[64]` / `Float[64]`): وقت البدء بالمللي ثانية. يقابل `t_start_ms`.
* `وقت_التحميل_بالمللي` / `tLoadMs` (`عـائم[64]` / `Float[64]`): وقت التحميل بالمللي ثانية. يقابل `t_load_ms`.
* `وقت_التقييم_الأولي_بالمللي` / `tPEvalMs` (`عـائم[64]` / `Float[64]`): وقت تقييم المحث. يقابل `t_p_eval_ms`.
* `وقت_التقييم_بالمللي` / `tEvalMs` (`عـائم[64]` / `Float[64]`): وقت تقييم الرموز. يقابل `t_eval_ms`.
* `عدد_التقييم_الأولي` / `nPEval` (`صـحيح` / `Int`): عدد تقييمات المحث. يقابل `n_p_eval`.
* `عدد_التقييم` / `nEval` (`صـحيح` / `Int`): عدد تقييمات الرموز. يقابل `n_eval`.
* `عدد_المعاد_استخدامه` / `nReused` (`صـحيح` / `Int`): عدد التقييمات المُعاد استخدامها. يقابل `n_reused`.

### بـيانات_معاين_أداء / PerfSamplerData

بيانات الأداء لعمليات المعاين. يقابل `llama_perf_sampler_data`.

* `وقت_المعاينة_بالمللي` / `tSampleMs` (`عـائم[64]` / `Float[64]`): وقت المعاينة بالمللي ثانية. يقابل `t_sample_ms`.
* `عدد_العينات` / `nSample` (`صـحيح` / `Int`): عدد العينات. يقابل `n_sample`.

## الأصناف

### نـموذج / Model

يمثل نموذج لغة كبير محمّل. يقابل `llama_model`.

#### نـموذج.مـعطيات / Model.Params

معطيات تحميل النموذج. يقابل `llama_model_params`.

* `الأجهزة` / `devices` (`مؤشر` / `ptr`): قائمة الأجهزة. يقابل `devices`.
* `استدراكات_نوع_صوان_الموتر` / `tensorBuftOverrides` (`سند[مصفوفة[اسـتدراك_نوع_صوان_موتر]]` / `ref[array[TensorBuftOverride]]`): استدراكات نوع صوان الموتر. يقابل `tensor_buft_overrides`.
* `عدد_طبقات_المعالج_الرسومي` / `nGpuLayers` (`صـحيح` / `Int`): عدد الطبقات المُفرَّغة لمعالج الرسوميات. يقابل `n_gpu_layers`.
* `نمط_التقسيم` / `splitMode` (`نـمط_تقسيم` / `SplitMode`): نمط تقسيم معالج الرسوميات. يقابل `split_mode`.
* `المعالج_الرسومي_الرئيسي` / `mainGpu` (`صـحيح` / `Int`): فهرس معالج الرسوميات الرئيسي. يقابل `main_gpu`.
* `تقسيم_الموتر` / `tensorSplit` (`سند[مصفوفة[عـائم]]` / `ref[array[Float]]`): نسب تقسيم الموتر. يقابل `tensor_split`.
* `دالة_التقدم` / `progressCallback` (`دالـة_تقدم` / `ProgressCallback`): دالة التقدم. يقابل `progress_callback`.
* `بيانات_دالة_التقدم` / `progressCallbackUserData` (`مؤشر` / `ptr`): بيانات المستخدم للدالة. يقابل `progress_callback_user_data`.
* `تجاوزات_مق` / `kvOverrides` (`مؤشر` / `ptr`): تجاوزات ذاكرة م-ق (مفتاح - قيمة). يقابل `kv_overrides`.
* `مفردات_فقط` / `vocabOnly` (`ثـنائي` / `Bool`): تحميل المفردات فقط. يقابل `vocab_only`.
* `استخدم_خريطة_ذاكرة` / `useMmap` (`ثـنائي` / `Bool`): استخدام خريطة الذاكرة. يقابل `use_mmap`.
* `استخدم_دخلا_مباشرا` / `useDirectIo` (`ثـنائي` / `Bool`): استخدام الإدخال/الإخراج المباشر. يقابل `use_direct_io`.
* `استخدم_قفل_ذاكرة` / `useMlock` (`ثـنائي` / `Bool`): قفل الذاكرة. يقابل `use_mlock`.
* `تفحص_الموترات` / `checkTensors` (`ثـنائي` / `Bool`): التحقق من بيانات الموترات. يقابل `check_tensors`.
* `استخدم_أنواع_صوان_إضافية` / `useExtraBufts` (`ثـنائي` / `Bool`): استخدام أنواع صوان إضافية. يقابل `use_extra_bufts`.
* `بلا_مضيف` / `noHost` (`ثـنائي` / `Bool`): بدون تخصيص صوان المضيف. يقابل `no_host`.
* `بلا_حجز` / `noAlloc` (`ثـنائي` / `Bool`): بدون تخصيص ذاكرة. يقابل `no_alloc`.

#### هات_معطيات_افتراضية / getDefaultParams

```
دالة نـموذج.هات_معطيات_افتراضية(): مـعطيات
```

<div dir=ltr>

```
func Model.getDefaultParams(): Params
```

</div>

الحصول على معطيات النموذج الافتراضية. يقابل `llama_model_default_params`.

#### حمل / load

```
دالة نـموذج.حمل(مسار: مـؤشر_محارف، معطيات: مـعطيات): سند[نـموذج]
```

<div dir=ltr>

```
func Model.load(path: CharsPtr, params: Params): ref[Model]
```

</div>

تحميل النموذج من ملف. يقابل `llama_model_load_from_file`.

#### حمل_أجزاء / loadSplits

```
دالة نـموذج.حمل_أجزاء(مسارات: سند[مصفوفة[مـؤشر_محارف]]، عدد: طـبيعي، معطيات: مـعطيات): سند[نـموذج]
```

<div dir=ltr>

```
func Model.loadSplits(paths: ref[array[CharsPtr]], count: Word, params: Params): ref[Model]
```

</div>

تحميل النموذج من ملفات مجزأة. يقابل `llama_model_load_from_splits`.

#### حرر / free

```
دالة نـموذج.حرر(نموذج: سند[نـموذج])
```

<div dir=ltr>

```
func Model.free(model: ref[Model])
```

</div>

تحرير موارد النموذج. يقابل `llama_model_free`.

#### احفظ / save

```
دالة نموذج.احفظ(مسار: مـؤشر_محارف)
```

<div dir=ltr>

```
func model.save(path: CharsPtr)
```

</div>

حفظ النموذج في ملف. يقابل `llama_model_save_to_file`.

#### لديه_مرمز / hasEncoder

```
نموذج.لديه_مرمز: ثـنائي
```

<div dir=ltr>

```
model.hasEncoder: Bool
```

</div>

التحقق من وجود مرمز. يقابل `llama_model_has_encoder`.

#### لديه_فاك_ترميز / hasDecoder

```
نموذج.لديه_فاك_ترميز: ثـنائي
```

<div dir=ltr>

```
model.hasDecoder: Bool
```

</div>

التحقق من وجود فاك ترميز. يقابل `llama_model_has_decoder`.

#### عدد_سياقات_التدريب / nCtxTrain

```
نموذج.عدد_سياقات_التدريب: صـحيح
```

<div dir=ltr>

```
model.nCtxTrain: Int
```

</div>

حجم سياق التدريب. يقابل `llama_model_n_ctx_train`.

#### عدد_التضمينات / nEmbd

```
نموذج.عدد_التضمينات: صـحيح
```

<div dir=ltr>

```
model.nEmbd: Int
```

</div>

بُعد التضمين. يقابل `llama_model_n_embd`.

#### عدد_الطبقات / nLayer

```
نموذج.عدد_الطبقات: صـحيح
```

<div dir=ltr>

```
model.nLayer: Int
```

</div>

عدد الطبقات. يقابل `llama_model_n_layer`.

#### عدد_الرؤوس / nHead

```
نموذج.عدد_الرؤوس: صـحيح
```

<div dir=ltr>

```
model.nHead: Int
```

</div>

عدد رؤوس الانتباه. يقابل `llama_model_n_head`.

#### المفردات / vocab

```
نموذج.المفردات: سند[مـفردات]
```

<div dir=ltr>

```
model.vocab: ref[Vocab]
```

</div>

الحصول على المفردات. يقابل `llama_model_get_vocab`.

#### هات_بيانات_وصفية_كنص / metaValStr

```
دالة نموذج.هات_بيانات_وصفية_كنص(
    مفتاح: مـؤشر_محارف، صوان: مـؤشر_محارف، حجم_الصوان: طـبيعي_متكيف
): صـحيح
```

<div dir=ltr>

```
func model.metaValStr(key: CharsPtr, buf: CharsPtr, bufSize: ArchWord): Int
```

</div>

الحصول على قيمة بيانات وصفية كنص. يقابل `llama_model_meta_val_str`.

#### عدد_البيانات_الوصفية / metaCount

```
نموذج.عدد_البيانات_الوصفية: صـحيح
```

<div dir=ltr>

```
model.metaCount: Int
```

</div>

الحصول على عدد البيانات الوصفية. يقابل `llama_model_meta_count`.

#### مفتاح_بيانات_وصفية_بفهرس / metaKeyByIndex

```
دالة نموذج.مفتاح_بيانات_وصفية_بفهرس(
    فهرس: صـحيح، صوان: مـؤشر_محارف، حجم_الصوان: طـبيعي_متكيف
): صـحيح
```

<div dir=ltr>

```
func model.metaKeyByIndex(idx: Int, buf: CharsPtr, bufSize: ArchWord): Int
```

</div>

الحصول على مفتاح بيانات وصفية بالفهرس. يقابل `llama_model_meta_key_by_index`.

### سـياق / Context

سياق الاستدلال للنموذج. يقابل `llama_context`.

#### سـياق.مـعطيات / Context.Params

معطيات إنشاء السياق. يقابل `llama_context_params`.

* `رقم_السياق` / `nCtx` (`طـبيعي` / `Word`): حجم السياق (0 = استخدام الافتراضي). يقابل `n_ctx`.
* `حجم_الدفعة_الظاهري` / `nBatch` (`طـبيعي` / `Word`): حجم الدفعة المنطقي. يقابل `n_batch`.
* `حجم_الدفعة_الحقيقي` / `nUbatch` (`طـبيعي` / `Word`): حجم الدفعة الفعلي. يقابل `n_ubatch`.
* `أقصى_تسلسل` / `nSeqMax` (`طـبيعي` / `Word`): أقصى عدد تسلسلات. يقابل `n_seq_max`.
* `عدد_المسالك` / `nThreads` (`صـحيح` / `Int`): عدد المسالك للتوليد. يقابل `n_threads`.
* `عدد_مسالك_الدفعة` / `nThreadsBatch` (`صـحيح` / `Int`): عدد مسالك معالجة الدفعة. يقابل `n_threads_batch`.
* `نوع_تحجيم_تمد` / `ropeScalingType` (`نـوع_تحجيم_تمد` / `RopeScalingType`): نوع تحجيم RoPE. يقابل `rope_scaling_type`.
* `نوع_التجميع` / `poolingType` (`نـوع_تجميع` / `PoolingType`): نوع التجميع. يقابل `pooling_type`.
* `نوع_الانتباه` / `attentionType` (`نـوع_انتباه` / `AttentionType`): نوع الانتباه. يقابل `attention_type`.
* `نوع_الانتباه_الخاطف` / `flashAttnType` (`نـوع_انتباه_خاطف` / `FlashAttnType`): إعداد الانتباه الخاطف. يقابل `flash_attn`.
* `تردد_تمد_الأساسي` / `ropeFreqBase` (`عائم` / `Float`): تردد RoPE الأساسي. يقابل `rope_freq_base`.
* `مقياس_تردد_تمد` / `ropeFreqScale` (`عائم` / `Float`): مقياس تردد RoPE. يقابل `rope_freq_scale`.
* `عامل_استقراء_الخيط` / `yarnExtFactor` (`عائم` / `Float`): عامل استقراء YaRN. يقابل `yarn_ext_factor`.
* `عامل_انتباه_الخيط` / `yarnAttnFactor` (`عائم` / `Float`): عامل انتباه YaRN. يقابل `yarn_attn_factor`.
* `بيتا_خيط_سريع` / `yarnBetaFast` (`عائم` / `Float`): بيتا YaRN السريع. يقابل `yarn_beta_fast`.
* `بيتا_خيط_بطيء` / `yarnBetaSlow` (`عائم` / `Float`): بيتا YaRN البطيء. يقابل `yarn_beta_slow`.
* `سياق_خيط_أصلي` / `yarnOrigCtx` (`طـبيعي` / `Word`): حجم سياق YaRN الأصلي. يقابل `yarn_orig_ctx`.
* `عتبة_الرص` / `defragThold` (`عائم` / `Float`): عتبة إزالة التجزئة. يقابل `defrag_thold`.
* `دالة_التقييم` / `cbEval` (`مؤشر دالة` / `function ptr`): دالة التقييم. يقابل `cb_eval`.
* `بيانات_دالة_التقييم` / `cbEvalData` (`مؤشر` / `ptr`): بيانات مستخدم دالة التقييم. يقابل `cb_eval_user_data`.
* `نوع_م` / `typeK` (`Ggml.Type`): نوع بيانات ذاكرة م. يقابل `type_k`.
* `نوع_ق` / `typeV` (`Ggml.Type`): نوع بيانات ذاكرة ق. يقابل `type_v`.
* `دالة_الإلغاء` / `abortCb` (`مؤشر دالة` / `function ptr`): دالة الإلغاء. يقابل `abort_callback`.
* `بيانات_دالة_الإلغاء` / `abortCbData` (`مؤشر` / `ptr`): بيانات دالة الإلغاء. يقابل `abort_callback_data`.
* `التضمينات` / `embeddings` (`ثـنائي` / `Bool`): تفعيل إخراج التضمينات. يقابل `embeddings`.
* `تفريغ_متق` / `offloadKqv` (`ثـنائي` / `Bool`): تفريغ م-ت-ق لمعالج الرسوميات. يقابل `offload_kqv`.
* `بلا_أداء` / `noPerf` (`ثـنائي` / `Bool`): تعطيل عدادات الأداء. يقابل `no_perf`.
* `تفريغ_عمليات` / `opOffload` (`ثـنائي` / `Bool`): تفعيل تفريغ العمليات. يقابل `op_offload`.
* `ذنم_كاملة_الحجم` / `swaFull` (`ثـنائي` / `Bool`): انتباه النافذة المنزلقة الكامل. يقابل `swa_full`.
* `مق_موحد` / `kvUnified` (`ثـنائي` / `Bool`): ذاكرة م-ق موحدة. يقابل `kv_unified`.
* `المعاينات` / `samplers` (`سند[مصفوفة[إعـدادات_تسلسل_معاين]]` / `ref[array[SamplerSeqConfig]]`): معاينات لكل تسلسل. يقابل `samplers`.
* `عدد_المعاينات` / `nSamplers` (`طـبيعي_متكيف` / `ArchWord`): عدد المعاينات. يقابل `n_samplers`.

#### هات_معطيات_افتراضية / getDefaultParams

```
دالة سـياق.هات_معطيات_افتراضية(): مـعطيات
```

<div dir=ltr>

```
func Context.getDefaultParams(): Params
```

</div>

الحصول على معطيات السياق الافتراضية. يقابل `llama_context_default_params`.

#### هيئ_من_نموذج / initFromModel

```
دالة سـياق.هيئ_من_نموذج(نموذج: سند[نـموذج]، معطيات: مـعطيات): سند[سـياق]
```

<div dir=ltr>

```
func Context.initFromModel(model: ref[Model], params: Params): ref[Context]
```

</div>

إنشاء سياق من نموذج. يقابل `llama_init_from_model`.

#### حرر / free

```
دالة سـياق.حرر(سياق: سند[سـياق])
```

<div dir=ltr>

```
func Context.free(ctx: ref[Context])
```

</div>

تحرير موارد السياق. يقابل `llama_free`.

#### أرفق_تجميع_مسالك / attachThreadpool

```
دالة سياق.أرفق_تجميع_مسالك(تج: سند[Ggml.Threadpool]، تج_دفعة: سند[Ggml.Threadpool])
```

<div dir=ltr>

```
func ctx.attachThreadpool(tp: ref[Ggml.Threadpool], tpBatch: ref[Ggml.Threadpool])
```

</div>

إرفاق تجميع مسالك. يقابل `llama_attach_threadpool`.

#### افصل_تجميع_مسالك / detachThreadpool

```
دالة سياق.افصل_تجميع_مسالك()
```

<div dir=ltr>

```
func ctx.detachThreadpool()
```

</div>

فصل تجميع المسالك. يقابل `llama_detach_threadpool`.

#### هات_تسلسل_تضمينات / getEmbeddingsSeq

```
دالة سياق.هات_تسلسل_تضمينات(معرف_التسلسل: مـعرف_تسلسل): سند[مصفوفة[عائم]]
```

<div dir=ltr>

```
func ctx.getEmbeddingsSeq(seqId: SeqId): ref[array[Float]]
```

</div>

الحصول على تضمينات التسلسل. يقابل `llama_get_embeddings_seq`.

#### هات_النموذج / getModel

```
دالة سياق.هات_النموذج(): سند[نـموذج]
```

<div dir=ltr>

```
func ctx.getModel(): ref[Model]
```

</div>

الحصول على النموذج المرتبط. يقابل `llama_get_model`.

#### رقم_السياق / nCtx

```
سياق.رقم_السياق: طـبيعي
```

<div dir=ltr>

```
ctx.nCtx: Word
```

</div>

الحصول على حجم السياق. يقابل `llama_n_ctx`.

#### حجم_الدفعة_الظاهري / nBatch

```
سياق.حجم_الدفعة_الظاهري: طـبيعي
```

<div dir=ltr>

```
ctx.nBatch: Word
```

</div>

الحصول على حجم الدفعة المنطقي. يقابل `llama_n_batch`.

#### حجم_الدفعة_الحقيقي / nUbatch

`سياق.حجم_الدفعة_الحقيقي: طـبيعي`

<div dir=ltr>

```
ctx.nUbatch: Word
```

</div>

الحصول على حجم الدفعة الفعلي. يقابل `llama_n_ubatch`.

#### أقصى_تسلسل / nSeqMax

```
سياق.أقصى_تسلسل: طـبيعي
```

<div dir=ltr>

```
ctx.nSeqMax: Word
```

</div>

الحصول على أقصى تسلسلات. يقابل `llama_n_seq_max`.

#### هات_الذاكرة / getMemory

```
دالة سياق.هات_الذاكرة(): مؤشر[ذاكـرة]
```

<div dir=ltr>

```
func ctx.getMemory(): ptr[Memory]
```

</div>

الحصول على ذاكرة م-ق. يقابل `llama_get_memory`.

#### نوع_التجميع / poolingType

```
سياق.نوع_التجميع: نـوع_تجميع
```

<div dir=ltr>

```
ctx.poolingType: PoolingType
```

</div>

الحصول على نوع التجميع. يقابل `llama_pooling_type`.

#### رمز / encode

```
دالة سياق.رمز(دفعة: دفـعة): صـحيح
```

<div dir=ltr>

```
func ctx.encode(batch: Batch): Int
```

</div>

ترميز الدفعة (لنماذج المرمز). يقابل `llama_encode`.

#### فك_ترميز / decode

```
دالة سياق.فك_ترميز(دفعة: دفـعة): صـحيح
```

<div dir=ltr>

```
func ctx.decode(batch: Batch): Int
```

</div>

فك ترميز الدفعة. يقابل `llama_decode`.

#### هات_لوجتات_ث / getLogitsIth

```
دالة سياق.هات_لوجتات_ث(ث: صـحيح): سند[مصفوفة[عائم]]
```

<div dir=ltr>

```
func ctx.getLogitsIth(i: Int): ref[array[Float]]
```

</div>

الحصول على اللوجتات للرمز بالفهرس. يقابل `llama_get_logits_ith`.

#### هات_تضمينات_ث / getEmbeddingsIth

```
دالة سياق.هات_تضمينات_ث(ث: صـحيح): سند[مصفوفة[عائم]]
```

<div dir=ltr>

```
func ctx.getEmbeddingsIth(i: Int): ref[array[Float]]
```

</div>

الحصول على التضمينات للرمز بالفهرس. يقابل `llama_get_embeddings_ith`.

#### حجم_الحالة / stateSize

```
سياق.حجم_الحالة: طـبيعي_متكيف
```

<div dir=ltr>

```
ctx.stateSize: ArchWord
```

</div>

الحصول على حجم الحالة بالبايت. يقابل `llama_state_get_size`.

#### هات_بيانات_الحالة / stateGetData

```
دالة سياق.هات_بيانات_الحالة(وجهة: سند[مصفوفة[طـبيعي[8]]]، حجم: طـبيعي_متكيف): طـبيعي_متكيف
```

<div dir=ltr>

```
func ctx.stateGetData(dst: ref[array[Word[8]]], size: ArchWord): ArchWord
```

</div>

الحصول على بيانات الحالة. يقابل `llama_state_get_data`.

#### حدد_بيانات_الحالة / stateSetData

```
دالة سياق.حدد_بيانات_الحالة(مصدر: سند[مصفوفة[طـبيعي[8]]]، حجم: طـبيعي_متكيف): طـبيعي_متكيف
```

<div dir=ltr>

```
func ctx.stateSetData(src: ref[array[Word[8]]], size: ArchWord): ArchWord
```

</div>

تحديد بيانات الحالة. يقابل `llama_state_set_data`.

#### حمل_ملف_حالة / stateLoadFile

```
دالة سياق.حمل_ملف_حالة(
    مسار: مـؤشر_محارف، رموز_الخرج: سند[مصفوفة[رمـز]]،
    سعة: طـبيعي_متكيف، عدد_الخرج: سند[طـبيعي_متكيف]
): ثـنائي
```

<div dir=ltr>

```
func ctx.stateLoadFile(
    path: CharsPtr, tokensOut: ref[array[Token]],
    cap: ArchWord, outCount: ref[ArchWord]
): Bool
```

</div>

تحميل الحالة من ملف. يقابل `llama_state_load_file`.

#### احفظ_ملف_حالة / stateSaveFile

```
دالة سياق.احفظ_ملف_حالة(مسار: مـؤشر_محارف، رموز: سند[مصفوفة[رمـز]]، عدد: طـبيعي_متكيف): ثـنائي
```

<div dir=ltr>

```
func ctx.stateSaveFile(path: CharsPtr, tokens: ref[array[Token]], count: ArchWord): Bool
```

</div>

حفظ الحالة في ملف. يقابل `llama_state_save_file`.

#### حدد_موائم_تمر / setAdapterLora

```
دالة سياق.حدد_موائم_تمر(موائم: سند[مـوائم]، مقياس: عائم): صـحيح
```

<div dir=ltr>

```
func ctx.setAdapterLora(ad: ref[Adapter], scale: Float): Int
```

</div>

تطبيق موائم LoRA. يقابل `llama_set_adapter_lora`.

#### أزل_موائم_تمر / removeAdapterLora

```
دالة سياق.أزل_موائم_تمر(موائم: سند[مـوائم]): صـحيح
```

<div dir=ltr>

```
func ctx.removeAdapterLora(ad: ref[Adapter]): Int
```

</div>

إزالة موائم LoRA. يقابل `llama_rm_adapter_lora`.

#### صفر_موائم_تمر / clearAdapterLora

```
دالة سياق.صفر_موائم_تمر()
```

<div dir=ltr>

```
func ctx.clearAdapterLora()
```

</div>

مسح جميع موائمات LoRA. يقابل `llama_clear_adapter_lora`.

#### سياق_الأداء / perfContext

```
سياق.سياق_الأداء: بـيانات_سياق_أداء
```

<div dir=ltr>

```
ctx.perfContext: PerfContextData
```

</div>

الحصول على بيانات الأداء. يقابل `llama_perf_context`.

#### اطبع_سياق_الأداء / perfContextPrint

```
دالة سياق.اطبع_سياق_الأداء()
```

<div dir=ltr>

```
func ctx.perfContextPrint()
```

</div>

طباعة بيانات الأداء. يقابل `llama_perf_context_print`.

#### صفر_سياق_الأداء / perfContextReset

```
دالة سياق.صفر_سياق_الأداء()
```

<div dir=ltr>

```
func ctx.perfContextReset()
```

</div>

إعادة تعيين عدادات الأداء. يقابل `llama_perf_context_reset`.

### دفـعة / Batch

دفعة رموز للمعالجة. يقابل `llama_batch`.

* `عدد_الرموز` / `nTokens` (`صـحيح` / `Int`): عدد الرموز. يقابل `n_tokens`.
* `الرموز` / `token` (`سند[مصفوفة[رمـز]]` / `ref[array[Token]]`): معرفات الرموز. يقابل `token`.
* `التضمين` / `embd` (`سند[مصفوفة[عائم]]` / `ref[array[Float]]`): التضمينات (بديل للرموز). يقابل `embd`.
* `الموقع` / `pos` (`سند[مصفوفة[مـوقع]]` / `ref[array[Pos]]`): مواقع الرموز. يقابل `pos`.
* `عدد_معرفات_التسلسل` / `nSeqId` (`سند[مصفوفة[صـحيح]]` / `ref[array[Int]]`): عدد معرفات التسلسل لكل رمز. يقابل `n_seq_id`.
* `معرفات_التسلسل` / `seqId` (`سند[مصفوفة[سند[مصفوفة[مـعرف_تسلسل]]]]` / `ref[array[ref[array[SeqId]]]]`): معرفات التسلسل. يقابل `seq_id`.
* `الخرج` / `output` (`سند[مصفوفة[صـحيح[8]]]` / `ref[array[Int[8]]]`): أعلام الإخراج. يقابل `logits`.

#### هات_واحدا / getOne

```
دالة دفـعة.هات_واحدا(رموز: سند[مصفوفة[رمـز]]، عدد_الرموز: صـحيح): دفـعة
```

<div dir=ltr>

```
func Batch.getOne(tokens: ref[array[Token]], nTokens: Int): Batch
```

</div>

إنشاء دفعة من مصفوفة رموز. يقابل `llama_batch_get_one`.

#### هيئ / init

```
دالة دفـعة.هيئ(عدد_الرموز: صـحيح، تضمين: صـحيح، أقصى_تسلسل: صـحيح): دفـعة
```

<div dir=ltr>

```
func Batch.init(nTokens: Int, embd: Int, nSeqMax: Int): Batch
```

</div>

تهيئة دفعة فارغة. يقابل `llama_batch_init`.

### مـعاين / Sampler

معاين رموز للتوليد. يقابل `llama_sampler`.

#### مـعاين.مـعطيات_سلسلة / Sampler.ChainParams

معطيات سلسلة المعاين. يقابل `llama_sampler_chain_params`.

* `بلا_أداء` / `noPerf` (`ثـنائي` / `Bool`): تعطيل عدادات الأداء. يقابل `no_perf`.

#### هيئ / init

```
دالة مـعاين.هيئ(واجهة: مؤشر، سياق: مؤشر): سند[مـعاين]
```

<div dir=ltr>

```
func Sampler.init(iface: ptr, ctx: ptr): ref[Sampler]
```

</div>

تهيئة معاين مخصص. يقابل `llama_sampler_init`.

#### هيئ_سلسلة / chainInit

```
دالة مـعاين.هيئ_سلسلة(معطيات: مـعطيات_سلسلة): سند[مـعاين]
```

<div dir=ltr>

```
func Sampler.chainInit(params: ChainParams): ref[Sampler]
```

</div>

إنشاء سلسلة معاينات. يقابل `llama_sampler_chain_init`.

#### هيئ_جشع / initGreedy

```
دالة مـعاين.هيئ_جشع(): سند[مـعاين]
```

<div dir=ltr>

```
func Sampler.initGreedy(): ref[Sampler]
```

</div>

إنشاء معاين جشع. يقابل `llama_sampler_init_greedy`.

#### هيئ_منتشر / initDist

```
دالة مـعاين.هيئ_منتشر(بذرة: طـبيعي): سند[مـعاين]
```

<div dir=ltr>

```
func Sampler.initDist(seed: Word): ref[Sampler]
```

</div>

إنشاء معاين توزيع. يقابل `llama_sampler_init_dist`.

#### هيئ_أعلى_ع / initTopK

```
دالة مـعاين.هيئ_أعلى_ع(ع: صـحيح): سند[مـعاين]
```

<div dir=ltr>

```
func Sampler.initTopK(k: Int): ref[Sampler]
```

</div>

إنشاء معاين أعلى-ع. يقابل `llama_sampler_init_top_k`.

#### هيئ_أعلى_ن / initTopP

```
دالة مـعاين.هيئ_أعلى_ن(ن: عائم، أدنى_إبقاء: طـبيعي_متكيف): سند[مـعاين]
```

<div dir=ltr>

```
func Sampler.initTopP(p: Float, minKeep: ArchWord): ref[Sampler]
```

</div>

إنشاء معاين أعلى-ن (النواة). يقابل `llama_sampler_init_top_p`.

#### هيئ_جزاءات / initPenalties

```
دالة مـعاين.هيئ_جزاءات(
    جزاء_آخر_ن: صـحيح، جزاء_التكرار: عائم، جزاء_التردد: عائم، جزاء_الوجود: عائم
): سند[مـعاين]
```

<div dir=ltr>

```
func Sampler.initPenalties(
    penaltyLastN: Int, penaltyRepeat: Float, penaltyFreq: Float, penaltyPresent: Float
): ref[Sampler]
```

</div>

إنشاء معاين الجزاءات. يقابل `llama_sampler_init_penalties`.

* `جزاء_آخر_ن` / `penaltyLastN`: آخر ن رمز للمعاقبة (0 = تعطيل، -1 = حجم السياق)
* `جزاء_التكرار` / `penaltyRepeat`: جزاء التكرار (1.0 = معطل)
* `جزاء_التردد` / `penaltyFreq`: جزاء التردد (0.0 = معطل)
* `جزاء_الوجود` / `penaltyPresent`: جزاء الوجود (0.0 = معطل)

#### انسخ / clone

```
دالة مـعاين.انسخ(م: سند[مـعاين]): سند[مـعاين]
```

<div dir=ltr>

```
func Sampler.clone(s: ref[Sampler]): ref[Sampler]
```

</div>

نسخ المعاين. يقابل `llama_sampler_clone`.

#### حرر / free

```
دالة مـعاين.حرر(م: سند[مـعاين])
```

<div dir=ltr>

```
func Sampler.free(s: ref[Sampler])
```

</div>

تحرير موارد المعاين. يقابل `llama_sampler_free`.

#### صفر / reset

```
دالة معاين.صفر()
```

<div dir=ltr>

```
func sampler.reset()
```

</div>

إعادة تعيين حالة المعاين. يقابل `llama_sampler_reset`.

#### عاين / sample

```
دالة معاين.عاين(سياق: سند[سـياق]، فهرس: صـحيح): رمـز
```

<div dir=ltr>

```
func sampler.sample(ctx: ref[Context], idx: Int): Token
```

</div>

معاينة الرمز التالي. يقابل `llama_sampler_sample`.

#### الاسم / name

```
معاين.الاسم: مـؤشر_محارف
```

<div dir=ltr>

```
sampler.name: CharsPtr
```

</div>

الحصول على اسم المعاين. يقابل `llama_sampler_name`.

#### اقبل / accept

```
دالة معاين.اقبل(رمز: رمـز)
```

<div dir=ltr>

```
func sampler.accept(token: Token)
```

</div>

قبول الرمز للتتبع. يقابل `llama_sampler_accept`.

#### طبق / apply

```
دالة معاين.طبق(مرشحون: سند[مـصفوفة_بيانات_رموز])
```

<div dir=ltr>

```
func sampler.apply(cand: ref[TokenDataArray])
```

</div>

تطبيق المعاين على المرشحين. يقابل `llama_sampler_apply`.

#### أضف_للسلسلة / chainAdd

```
دالة معاين.أضف_للسلسلة(م: سند[مـعاين])
```

<div dir=ltr>

```
func sampler.chainAdd(s: ref[Sampler])
```

</div>

إضافة معاين للسلسلة. يقابل `llama_sampler_chain_add`.

#### هات_من_السلسلة / chainGet

```
دالة معاين.هات_من_السلسلة(فهرس: صـحيح): سند[مـعاين]
```

<div dir=ltr>

```
func sampler.chainGet(idx: Int): ref[Sampler]
```

</div>

الحصول على معاين من السلسلة. يقابل `llama_sampler_chain_get`.

#### عدد_السلسلة / chainCount

```
معاين.عدد_السلسلة: صـحيح
```

<div dir=ltr>

```
sampler.chainCount: Int
```

</div>

الحصول على طول السلسلة. يقابل `llama_sampler_chain_n`.

#### أزل_من_السلسلة / chainRemove

```
دالة معاين.أزل_من_السلسلة(فهرس: صـحيح): سند[مـعاين]
```

<div dir=ltr>

```
func sampler.chainRemove(idx: Int): ref[Sampler]
```

</div>

إزالة معاين من السلسلة. يقابل `llama_sampler_chain_remove`.

#### معاين_الأداء / perfSampler

```
معاين.معاين_الأداء: بـيانات_معاين_أداء
```

<div dir=ltr>

```
sampler.perfSampler: PerfSamplerData
```

</div>

الحصول على بيانات الأداء. يقابل `llama_perf_sampler`.

#### اطبع_معاين_الأداء / perfSamplerPrint

```
دالة معاين.اطبع_معاين_الأداء()
```

<div dir=ltr>

```
func sampler.perfSamplerPrint()
```

</div>

طباعة بيانات الأداء. يقابل `llama_perf_sampler_print`.

#### صفر_معاين_الأداء / perfSamplerReset

```
دالة معاين.صفر_معاين_الأداء()
```

<div dir=ltr>

```
func sampler.perfSamplerReset()
```

</div>

إعادة تعيين عدادات الأداء. يقابل `llama_perf_sampler_reset`.

### مـفردات / Vocab

مفردات النموذج. يقابل `llama_vocab`.

#### رمز_إلى_قطعة / tokenToPiece

```
دالة مفردات.رمز_إلى_قطعة(رمز: رمـز، صوان: مـؤشر_محارف، طول: صـحيح، قص_يسار: صـحيح، خاص: ثـنائي): صـحيح
```

<div dir=ltr>

```
func vocab.tokenToPiece(token: Token, buf: CharsPtr, length: Int, lstrip: Int, special: Bool): Int
```

</div>

تحويل الرمز إلى نص. يقابل `llama_token_to_piece`.

#### نهاية_الجملة / eos

```
مفردات.نهاية_الجملة: رمـز
```

<div dir=ltr>

```
vocab.eos: Token
```

</div>

الحصول على رمز نهاية التسلسل. يقابل `llama_vocab_eos`.

#### أهي_نهاية_التوليد / isEog

```
دالة مفردات.أهي_نهاية_التوليد(رمز: رمـز): ثـنائي
```

<div dir=ltr>

```
func vocab.isEog(token: Token): Bool
```

</div>

التحقق إن كان الرمز نهاية التوليد. يقابل `llama_vocab_is_eog`.

### ذاكـرة / Memory

إدارة ذاكرة م-ق. يقابل `llama_memory`.

#### صفر / clear

```
دالة ذاكرة.صفر(بيانات: ثـنائي)
```

<div dir=ltr>

```
func memory.clear(data: Bool)
```

</div>

مسح ذاكرة م-ق. يقابل `llama_memory_clear`.

#### أزل_تسلسلا / seqRemove

```
دالة ذاكرة.أزل_تسلسلا(تسلسل: مـعرف_تسلسل، م0: مـوقع، م1: مـوقع): ثـنائي
```

<div dir=ltr>

```
func memory.seqRemove(seq: SeqId, p0: Pos, p1: Pos): Bool
```

</div>

إزالة نطاق تسلسل. يقابل `llama_memory_seq_rm`.

#### انسخ_تسلسلا / seqCopy

```
دالة ذاكرة.انسخ_تسلسلا(مصدر: مـعرف_تسلسل، وجهة: مـعرف_تسلسل، م0: مـوقع، م1: مـوقع)
```

<div dir=ltr>

```
func memory.seqCopy(src: SeqId, dst: SeqId, p0: Pos, p1: Pos)
```

</div>

نسخ نطاق تسلسل. يقابل `llama_memory_seq_cp`.

#### أبق_تسلسلا / seqKeep

```
دالة ذاكرة.أبق_تسلسلا(تسلسل: مـعرف_تسلسل)
```

<div dir=ltr>

```
func memory.seqKeep(seq: SeqId)
```

</div>

الإبقاء على التسلسل المحدد فقط. يقابل `llama_memory_seq_keep`.

#### أضف_لتسلسل / seqAdd

```
دالة ذاكرة.أضف_لتسلسل(تسلسل: مـعرف_تسلسل، م0: مـوقع، م1: مـوقع، فرق: مـوقع)
```

<div dir=ltr>

```
func memory.seqAdd(seq: SeqId, p0: Pos, p1: Pos, delta: Pos)
```

</div>

إضافة فرق موقع للتسلسل. يقابل `llama_memory_seq_add`.

#### قسم_تسلسلا / seqDiv

```
دالة ذاكرة.قسم_تسلسلا(تسلسل: مـعرف_تسلسل، م0: مـوقع، م1: مـوقع، ق: صـحيح)
```

<div dir=ltr>

```
func memory.seqDiv(seq: SeqId, p0: Pos, p1: Pos, d: Int)
```

</div>

تقسيم المواقع في التسلسل. يقابل `llama_memory_seq_div`.

### مـوائم / Adapter

دعم موائم LoRA. يقابل `llama_adapter_lora`.

#### تمر_هيئ / loraInit

```
دالة مـوائم.تمر_هيئ(نموذج: سند[نـموذج]، مسار: مـؤشر_محارف): سند[مـوائم]
```

<div dir=ltr>

```
func Adapter.loraInit(model: ref[Model], path: CharsPtr): ref[Adapter]
```

</div>

تحميل موائم LoRA. يقابل `llama_adapter_lora_init`.

#### تمر_حرر / loraFree

```
دالة مـوائم.تمر_حرر(موائم: سند[مـوائم])
```

<div dir=ltr>

```
func Adapter.loraFree(ad: ref[Adapter])
```

</div>

تحرير موائم LoRA. يقابل `llama_adapter_lora_free`.

## الدالات العمومية

### إدارة المشغل

#### هيئ_المشغل / backendInit

```
دالة هيئ_المشغل()
```

<div dir=ltr>

```
func backendInit()
```

</div>

تهيئة مشغل لاما. يقابل `llama_backend_init`.

#### حرر_المشغل / backendFree

```
دالة حرر_المشغل()
```

<div dir=ltr>

```
func backendFree()
```

</div>

تحرير مشغل لاما. يقابل `llama_backend_free`.

#### هيئ_نوما / numaInit

```
دالة هيئ_نوما(استراتيجية: Ggml.NumaStrategy)
```

<div dir=ltr>

```
func numaInit(strategy: Ggml.NumaStrategy)
```

</div>

تهيئة NUMA. يقابل `llama_numa_init`.

### معلومات النظام

#### الوقت_بالميكرو / timeUs

```
دالة الوقت_بالميكرو(): صـحيح[64]
```

<div dir=ltr>

```
func timeUs(): Int[64]
```

</div>

الحصول على الوقت الحالي بالميكروثانية. يقابل `llama_time_us`.

#### أقصى_أجهزة / maxDevices

```
دالة أقصى_أجهزة(): طـبيعي_متكيف
```

<div dir=ltr>

```
func maxDevices(): ArchWord
```

</div>

الحصول على أقصى عدد أجهزة. يقابل `llama_max_devices`.

#### أقصى_تسلسلات_متوازية / maxParallelSequences

```
دالة أقصى_تسلسلات_متوازية(): طـبيعي_متكيف
```

<div dir=ltr>

```
func maxParallelSequences(): ArchWord
```

</div>

الحصول على أقصى تسلسلات متوازية. يقابل `llama_max_parallel_sequences`.

#### يدعم_خريطة_الذاكرة / supportsMmap

```
دالة يدعم_خريطة_الذاكرة(): ثـنائي
```

<div dir=ltr>

```
func supportsMmap(): Bool
```

</div>

التحقق من دعم خريطة الذاكرة. يقابل `llama_supports_mmap`.

#### يدعم_قفل_الذاكرة / supportsMlock

```
دالة يدعم_قفل_الذاكرة(): ثـنائي
```

<div dir=ltr>

```
func supportsMlock(): Bool
```

</div>

التحقق من دعم قفل الذاكرة. يقابل `llama_supports_mlock`.

#### يدعم_تفويض_معالج_رسومي / supportsGpuOffload

```
دالة يدعم_تفويض_معالج_رسومي(): ثـنائي
```

<div dir=ltr>

```
func supportsGpuOffload(): Bool
```

</div>

التحقق من دعم التفويض لمعالج الرسوميات. يقابل `llama_supports_gpu_offload`.

#### يدعم_استدعاء_بعيدا / supportsRpc

```
دالة يدعم_استدعاء_بعيدا(): ثـنائي
```

<div dir=ltr>

```
func supportsRpc(): Bool
```

</div>

التحقق من دعم RPC. يقابل `llama_supports_rpc`.

#### اطبع_معلومات_النظام / printSystemInfo

```
دالة اطبع_معلومات_النظام(): مـؤشر_محارف
```

<div dir=ltr>

```
func printSystemInfo(): CharsPtr
```

</div>

الحصول على نص معلومات النظام. يقابل `llama_print_system_info`.

### الترميز

#### رمز / tokenize

```
دالة رمز(
    مفردات: سند[مـفردات]، نص: مـؤشر_محارف، طول_النص: صـحيح، رموز: سند[مصفوفة[رمـز]]،
    أقصى_رموز: صـحيح، أضف_خاصة: ثـنائي، حلل_خاصة: ثـنائي
): صـحيح
```

<div dir=ltr>

```
func tokenize(
    vocab: ref[Vocab], text: CharsPtr, textLen: Int, tokens: ref[array[Token]],
    nTokensMax: Int, addSpecial: Bool, parseSpecial: Bool
): Int
```

</div>

تحويل النص إلى رموز. يُرجع عدد الرموز المكتوبة. يقابل `llama_tokenize`.

#### فك_ترميز / detokenize

```
دالة فك_ترميز(
    مفردات: سند[مـفردات]، رموز: سند[مصفوفة[رمـز]]، عدد_الرموز: صـحيح، نص: مـؤشر_محارف،
    أقصى_طول_نص: صـحيح، أزل_خاصة: ثـنائي، ألغ_تحليل_خاصة: ثـنائي
): صـحيح
```

<div dir=ltr>

```
func detokenize(
    vocab: ref[Vocab], tokens: ref[array[Token]], nTokens: Int, text: CharsPtr,
    textLenMax: Int, removeSpecial: Bool, unparseSpecial: Bool
): Int
```

</div>

تحويل الرموز إلى نص. يُرجع عدد المحارف المكتوبة. يقابل `llama_detokenize`.

### قوالب المحادثة

#### طبق_قالب_محادثة / chatApplyTemplate

```
دالة طبق_قالب_محادثة(
    قالب: مـؤشر_محارف، محادثة: سند[مصفوفة[رسـالة_محادثة]]، عدد_الرسائل: طـبيعي_متكيف،
    أضف_مساعدا: ثـنائي، صوان: مـؤشر_محارف، طول_الصوان: صـحيح
): صـحيح
```

<div dir=ltr>

```
func chatApplyTemplate(
    tmpl: CharsPtr, chat: ref[array[ChatMessage]], nMsg: ArchWord,
    addAssistant: Bool, buf: CharsPtr, bufLen: Int
): Int
```

</div>

تطبيق قالب المحادثة على الرسائل. مرر `0` لـ `قالب` / `tmpl` لاستخدام قالب النموذج الافتراضي. يُرجع عدد المحارف المكتوبة. يقابل `llama_chat_apply_template`.

#### قوالب_محادثة_مدمجة / chatBuiltinTemplates

```
دالة قوالب_محادثة_مدمجة(خرج: سند[مـؤشر_محارف]، طول: طـبيعي_متكيف): صـحيح
```

<div dir=ltr>

```
func chatBuiltinTemplates(out: ref[CharsPtr], len: ArchWord): Int
```

</div>

الحصول على أسماء القوالب المدمجة. يقابل `llama_chat_builtin_templates`.

### التسجيل

#### حدد_دالة_تسجيل / logSet

```
دالة حدد_دالة_تسجيل(
    دالة: مؤشر[دالة (مستوى: صـحيح، نص: مـؤشر_محارف، بيانات_المستخدم: مؤشر)]،
    بيانات_المستخدم: مؤشر
)
```

<div dir=ltr>

```
func logSet(
    cb: ptr[function(level: Int, text: CharsPtr, userData: ptr)],
    userData: ptr
)
```

</div>

تحديد دالة التسجيل. يقابل `llama_log_set`.

## الأمثلة

### إكمال النص

انظر `Examples/إكمال.أسس` أو `Examples/completion.alusus` لمثال كامل على إكمال النص.

### المحادثة

انظر `Examples/محادثة.أسس` أو `Examples/chat.alusus` لمثال محادثة متعددة الأدوار مع قوالب المحادثة.

### نمط الاستخدام الأساسي / Basic Usage Pattern

```
اشمل "مـحا"؛
مـحا.اشمل_حزمة("Alusus/Llama@0.2"، "لـاما.أسس")؛
استخدم لـاما؛

// ١. تهيئة المشغل
جـجمل.مـشغل.حمل_للمعالج()؛

// ٢. تحميل النموذج
عرف معطيات_نموذج: نـموذج.مـعطيات = نـموذج.هات_معطيات_افتراضية()؛
معطيات_نموذج.عدد_طبقات_المعالج_الرسومي = 0؛  // المعالج المركزي فقط
عرف نموذج: سند[نـموذج](نـموذج.حمل("model.gguf"، معطيات_نموذج))؛

// ٣. إنشاء السياق
عرف معطيات_سياق: سـياق.مـعطيات = سـياق.هات_معطيات_افتراضية()؛
معطيات_سياق.رقم_السياق = 2048؛
عرف سياق: سند[سـياق](سـياق.هيئ_من_نموذج(نموذج، معطيات_سياق))؛

// ٤. إنشاء سلسلة معاينات
عرف معطيات_سلسلة: مـعاين.مـعطيات_سلسلة؛
معطيات_سلسلة.بلا_أداء = 1؛
عرف معاين: سند[مـعاين](مـعاين.هيئ_سلسلة(معطيات_سلسلة))؛
معاين.أضف_للسلسلة(مـعاين.هيئ_جزاءات(64، 1.1، 0.0، 0.0))؛
معاين.أضف_للسلسلة(مـعاين.هيئ_أعلى_ع(40))؛
معاين.أضف_للسلسلة(مـعاين.هيئ_أعلى_ن(0.9، 1))؛
معاين.أضف_للسلسلة(مـعاين.هيئ_منتشر(0))؛

// ٥. ترميز المحث وفك ترميزه
عرف رموز: مصفوفة[رمـز، 512]؛
عرف عدد_الرموز: صـحيح = رمز(نموذج.المفردات، "مرحبا أيها العالم"، 34، رموز، 512، 1، 1)؛
عرف دفعة: دفـعة = دفـعة.هات_واحدا(رموز، عدد_الرموز)؛
سياق.فك_ترميز(دفعة)؛

// ٦. توليد الرموز
بينما صحيح {
    عرف معرف: رمـز = معاين.عاين(سياق، -1)؛
    إن نموذج.المفردات.أهي_نهاية_التوليد(معرف) أكسر؛

    // تحويل الرمز إلى نص وطباعته
    عرف صوان: مصفوفة[محرف، 64]؛
    فك_ترميز(نموذج.المفردات، معرف~مؤشر~تحويل[سند[مصفوفة[رمـز]]]، 1، صوان~مؤشر، 64، 0، 0)؛
    // ... اطبع الصوان

    // فك ترميز الرمز التالي
    عرف دفعة_تالية: دفـعة = دفـعة.هات_واحدا(معرف~مؤشر~تحويل[سند[مصفوفة[رمـز]]]، 1)؛
    سياق.فك_ترميز(دفعة_تالية)؛
}

// ٧. التنظيف
مـعاين.حرر(معاين)؛
سـياق.حرر(سياق)؛
نـموذج.حرر(نموذج)؛
```

<div dir=ltr>

```
import "Apm";
Apm.importPackage("Alusus/Llama@0.2");
use Llama;

// 1. Initialize backend
Ggml.Backend.cpuLoad();

// 2. Load model
def modelParams: Model.Params = Model.getDefaultParams();
modelParams.nGpuLayers = 0;  // CPU only
def model: ref[Model](Model.load("model.gguf", modelParams));

// 3. Create context
def ctxParams: Context.Params = Context.getDefaultParams();
ctxParams.nCtx = 2048;
def ctx: ref[Context](Context.initFromModel(model, ctxParams));

// 4. Create sampler chain
def chainParams: Sampler.ChainParams;
chainParams.noPerf = true;
def sampler: ref[Sampler](Sampler.chainInit(chainParams));
sampler.chainAdd(Sampler.initPenalties(64, 1.1, 0.0, 0.0));
sampler.chainAdd(Sampler.initTopK(40));
sampler.chainAdd(Sampler.initTopP(0.9, 1));
sampler.chainAdd(Sampler.initDist(0));

// 5. Tokenize and decode prompt
def tokens: array[Token, 512];
def nTokens: Int = tokenize(model.vocab, "Hello world", 11, tokens, 512, true, true);
def batch: Batch = Batch.getOne(tokens, nTokens);
ctx.decode(batch);

// 6. Generate tokens
while true {
    def id: Token = sampler.sample(ctx, -1);
    if model.vocab.isEog(id) break;

    // Convert token to text and print
    def buf: array[Char, 64];
    detokenize(model.vocab, id~ptr~cast[ref[array[Token]]], 1, buf~ptr, 64, 0, 0);
    // ... print buf

    // Decode next token
    def nextBatch: Batch = Batch.getOne(id~ptr~cast[ref[array[Token]]], 1);
    ctx.decode(nextBatch);
}

// 7. Cleanup
Sampler.free(sampler);
Context.free(ctx);
Model.free(model);
```

</div>

## الرخصة

حقوق النشر (c) 2023-2024 The ggml authors
حقوق النشر (c) 2026 شركة أُلوسُس للبرمجيات للروابط باللغة الأسسية.

تتبع هذه المكتبة نفس رخصة llama.cpp (رخصة MIT).

</div>
