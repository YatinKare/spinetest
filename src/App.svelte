<script>
  import { tick } from 'svelte';

  let language = $state('ko');
  let activeTab = $state('measure');
  let cobb = $state(0);
  let curveDirection = $state('right');
  let image = $state(null);
  let imageSize = $state({ width: 0, height: 0 });
  let points = $state([]);
  let atr = $state(0);
  let toast = $state('');
  let toastTimer;
  let canvas = $state(null);

  const copy = {
    ko: {
      title: 'SpineCare BioLab', badge: '자가 관리 솔루션', language: 'English',
      tabs: { measure: '아담스 검사 측정', severity: '중증도 분석', exercise: '영상 운동 가이드' },
      measureTitle: '아담스 전굴 검사 (사진 각도 측정)',
      measureHelp: '허리를 90도로 숙여 등고선이 잘 보이게 찍은 사진을 업로드하세요. 사진 위에서 등의 가장 높이 솟은 두 곳(좌측 1번, 우측 1번)을 클릭하면 각도가 자동 측정됩니다.',
      upload: '아담스 테스트 사진 업로드', choose: '사진 선택하기', reset: '점 다시 찍기',
      atr: '측정된 등 회전 각도 (ATR)', analyze: '이 각도로 중증도 분석하기',
      severityTitle: '임상적 중증도 및 척추 역학 분석', cobb: '추정 콥스 앵글 (Cobb’s Angle)',
      estimate: '* ATR 각도에 기반한 추정치입니다.', curve: '커브 변형 방향 (튀어나온 등 위치)',
      right: '우측 등 돌출 (Right Convexity)', left: '좌측 등 돌출 (Left Convexity)',
      exerciseTitle: '4단계 맞춤형 운동 루틴',
      exerciseIntro: '의학 가이드라인 기반의 자가 교정 시퀀스입니다. 영상을 보며 구체적인 신체 감각을 따라 하세요. (※ 기구는 척추를 강제로 펴주지 않습니다. 본인의 인지가 가장 중요합니다.)',
      warning: '이 도구는 교육 및 자가 관찰용이며 진단을 제공하지 않습니다. Cobb 각도는 의료 영상과 전문 평가로 확인하세요.',
      status: ['대칭', '미세 불균형', '측만증 의심 (관찰 필요)', '측만증 정밀진단 권장'],
      normal: '정상', mild: '경도 (Mild)', advanced: '중등도 이상',
      normalTitle: '미세 척추 정렬', mildTitle: '경도 척추측만', advancedTitle: '진행성 척추측만',
      normalDesc: '임상적 측만증에 해당하지 않습니다. 양측 대칭을 맞추는 일반적인 코어 강화 및 바른 자세 유지가 권장됩니다.',
      mildDesc: '주기적인 관찰이 필요합니다. 오목해진 흉곽을 확장시키는 3차원 호흡법과 단축된 근육을 늘리는 운동을 진행하세요.',
      advancedDesc: '보조기 착용 및 전문의 상담이 필요한 단계입니다. 아래 운동은 전문가의 지도 하에 보조적으로만 수행하십시오.',
      placeholder: '데이터를 입력해주세요', placeholderDesc: '측정된 각도를 기반으로 척추의 생체역학적 불균형 상태를 분석합니다.',
      steps: [
        { title: '척추 길이 인지 및 호흡 (Wall Bars 활용)', phase: '인지 단계', video: 'https://www.youtube.com/embed/n4Kj72F7Yx0', videoTitle: '호흡법 가이드', how: '월바나 방문틀을 양손으로 잡고 발을 고정한 채 엉덩이를 뒤로 살짝 뺍니다. 정수리를 천장 쪽으로 뽑아내듯 척추를 길게 늘립니다.', focus: '숨을 들이마실 때 움푹 파인 {concave} 등으로 공기를 채워 풍선처럼 부풀어 오르는 감각에 집중하세요.', note: '기구에 매달려 허리를 꺾지 마세요. 기구는 자세를 고정하는 용도입니다.' },
        { title: '오목측 타겟 이완 (밴드/매트 활용)', phase: '이완 단계', video: 'https://www.youtube.com/embed/J72_z2t6zT8', videoTitle: '스트레칭 가이드', how: '바닥에 앉아 밴드를 발에 걸고, {concave} 옆구리가 늘어나도록 상체를 반대편({convex})으로 지그시 기울입니다.', focus: '{concave} 갈비뼈 사이가 벌어지는 감각을 느끼며 밴드의 장력을 유지하고 15초간 머뭅니다.' },
        { title: '볼록측 등근육 강화 (저항 밴드 로우)', phase: '강화 단계', video: 'https://www.youtube.com/embed/WJ_6A7E8e7k', videoTitle: '로우 밴드 가이드', how: '밴드를 기둥에 묶고 {convex} 팔로 골반 쪽을 향해 당겨 날개뼈 주변 근육을 강화합니다.', focus: '어깨를 으쓱하지 않고 {convex} 날개뼈가 척추 쪽으로 모이는 감각에 집중하세요.' },
        { title: '골반 안정화 및 코어 균형 (짐볼/매트 브릿지)', phase: '균형 단계', video: 'https://www.youtube.com/embed/oQ3-87D_Ea0', videoTitle: '브릿지 가이드', how: '천장을 보고 누워 양발을 골반 너비로 벌립니다. 발바닥으로 바닥을 밀어내며 엉덩이를 들어 올립니다.', focus: '허리를 과하게 꺾지 말고 양쪽 엉덩이에 같은 힘이 들어가는지 확인하며 골반을 평행하게 유지하세요.' }
      ]
    },
    en: {
      title: 'SpineCare BioLab', badge: 'Self-care guide', language: '한국어',
      tabs: { measure: 'Adams Test', severity: 'Severity', exercise: 'Exercise Videos' },
      measureTitle: 'Adams forward bend test (photo angle measurement)',
      measureHelp: 'Upload a photo taken while bending forward about 90 degrees, with the back contours visible. Click the two highest points on the back—one on each side—to estimate the angle.',
      upload: 'Upload an Adams test photo', choose: 'Choose photo', reset: 'Reset points',
      atr: 'Estimated angle of trunk rotation (ATR)', analyze: 'Analyze severity with this angle',
      severityTitle: 'Clinical severity and spinal mechanics', cobb: 'Estimated Cobb angle',
      estimate: '* This is an estimate based on the ATR angle.', curve: 'Curve direction (side of back prominence)',
      right: 'Right back prominence (right convexity)', left: 'Left back prominence (left convexity)',
      exerciseTitle: 'A four-step exercise routine',
      exerciseIntro: 'A self-guided sequence informed by medical guidance. Follow the movement cues as you watch. Equipment does not force the spine straight; body awareness matters most.',
      warning: 'For education and self-observation only; this tool does not diagnose. Cobb angle should be assessed from medical imaging by a qualified clinician.',
      status: ['Symmetric', 'Minor asymmetry', 'Possible scoliosis (monitor)', 'Consider clinical evaluation'],
      normal: 'Within typical range', mild: 'Mild', advanced: 'Moderate or greater',
      normalTitle: 'Minimal spinal curvature', mildTitle: 'Mild scoliosis range', advancedTitle: 'Higher curvature range',
      normalDesc: 'This is below the clinical threshold for scoliosis. General core strengthening and good posture may support balanced movement.',
      mildDesc: 'Periodic monitoring may be appropriate. Consider three-dimensional breathing and mobility work with guidance from a clinician.',
      advancedDesc: 'A clinician can discuss bracing and other care options. Use the exercises below only as supportive activity under professional guidance.',
      placeholder: 'Enter an angle to see an estimate', placeholderDesc: 'The estimate is based on the entered angle and is not a diagnosis.',
      steps: [
        { title: 'Spinal length and breathing (wall bars)', phase: 'Awareness', video: 'https://www.youtube.com/embed/n4Kj72F7Yx0', videoTitle: 'Breathing guide', how: 'Hold wall bars or a door frame with both hands. Keep your feet planted and gently move your hips back. Imagine lengthening your spine toward the ceiling.', focus: 'As you inhale, focus on filling the hollow {concave} side of your back with air, like a balloon expanding.', note: 'Do not hang from equipment or arch your lower back. Use it only to steady your posture.' },
        { title: 'Targeted release on the concave side (band/mat)', phase: 'Mobility', video: 'https://www.youtube.com/embed/J72_z2t6zT8', videoTitle: 'Stretching guide', how: 'Sit with a band around your foot and gently lean toward the {convex} side to stretch your shortened {concave} side.', focus: 'Maintain gentle band tension and hold the sensation of space opening between the ribs for 15 seconds.' },
        { title: 'Strengthen back muscles on the convex side (band row)', phase: 'Strength', video: 'https://www.youtube.com/embed/WJ_6A7E8e7k', videoTitle: 'Resistance band row guide', how: 'Anchor a band and pull toward your hip with the {convex} arm to strengthen the muscles around that shoulder blade.', focus: 'Keep your shoulder relaxed as you pull and feel the {convex} shoulder blade draw toward the spine.' },
        { title: 'Pelvic stability and core balance (bridge)', phase: 'Balance', video: 'https://www.youtube.com/embed/oQ3-87D_Ea0', videoTitle: 'Bridge guide', how: 'Lie on your back with feet hip-width apart. Press through your feet to lift your hips.', focus: 'Avoid over-arching your lower back. Keep your pelvis level and check that both glutes work evenly.' }
      ]
    }
  };

  let t = $derived(copy[language]);
  let convexSide = $derived(curveDirection === 'right' ? (language === 'ko' ? '우측' : 'right') : (language === 'ko' ? '좌측' : 'left'));
  let concaveSide = $derived(curveDirection === 'right' ? (language === 'ko' ? '좌측' : 'left') : (language === 'ko' ? '우측' : 'right'));
  let severity = $derived(cobb < 10 ? 'normal' : cobb <= 20 ? 'mild' : 'advanced');
  let severityCopy = $derived({
    normal: { label: t.normal, title: t.normalTitle, desc: t.normalDesc },
    mild: { label: t.mild, title: t.mildTitle, desc: t.mildDesc },
    advanced: { label: t.advanced, title: t.advancedTitle, desc: t.advancedDesc }
  }[severity]);
  let atrStatus = $derived(atr === 0 ? t.status[0] : atr <= 5 ? t.status[1] : atr <= 10 ? t.status[2] : t.status[3]);

  function notify(message) {
    toast = message;
    clearTimeout(toastTimer);
    toastTimer = setTimeout(() => toast = '', 3200);
  }

  function exerciseText(value) {
    return value.replaceAll('{concave}', concaveSide).replaceAll('{convex}', convexSide);
  }

  function loadPhoto(event) {
    const file = event.currentTarget.files?.[0];
    if (!file) return;
    const reader = new FileReader();
    reader.onload = async () => {
      const nextImage = new Image();
      nextImage.onload = async () => {
        image = nextImage;
        const width = Math.min(760, Math.max(280, document.querySelector('.canvas-frame')?.clientWidth ?? 680));
        imageSize = { width, height: Math.round(nextImage.height * width / nextImage.width) };
        points = [];
        atr = 0;
        await tick();
        drawPhoto();
      };
      nextImage.src = reader.result;
    };
    reader.readAsDataURL(file);
  }

  function drawPhoto() {
    if (!canvas || !image) return;
    const context = canvas.getContext('2d');
    context.clearRect(0, 0, canvas.width, canvas.height);
    context.drawImage(image, 0, 0, canvas.width, canvas.height);
    for (const point of points) {
      context.beginPath();
      context.arc(point.x, point.y, 7, 0, Math.PI * 2);
      context.fillStyle = '#ef4444';
      context.fill();
      context.lineWidth = 2;
      context.strokeStyle = '#fff';
      context.stroke();
    }
    if (points.length === 2) {
      context.beginPath();
      context.moveTo(points[0].x, points[0].y);
      context.lineTo(points[1].x, points[1].y);
      context.strokeStyle = '#2563eb';
      context.lineWidth = 3;
      context.stroke();
    }
  }

  function clickCanvas(event) {
    if (points.length >= 2 || !canvas) return;
    const rect = canvas.getBoundingClientRect();
    const point = { x: (event.clientX - rect.left) * canvas.width / rect.width, y: (event.clientY - rect.top) * canvas.height / rect.height };
    points = [...points, point];
    if (points.length === 2) {
      const [first, second] = points;
      const angle = Math.atan2(Math.abs(second.y - first.y), Math.abs(second.x - first.x)) * 180 / Math.PI;
      atr = Math.min(Math.round(angle * 10) / 10, 30);
      const leftPoint = first.x < second.x ? first : second;
      const rightPoint = first.x > second.x ? first : second;
      curveDirection = rightPoint.y < leftPoint.y ? 'right' : 'left';
      notify(`${atr}° ${language === 'ko' ? '각도로 측정되었습니다.' : 'estimated from the selected points.'}`);
    }
    drawPhoto();
  }

  function resetPoints() {
    points = [];
    atr = 0;
    drawPhoto();
  }

  function analyzeFromPhoto() {
    if (atr === 0) {
      notify(language === 'ko' ? '먼저 사진에 두 점을 클릭하여 각도를 측정해주세요.' : 'Select two points on the photo to estimate an angle first.');
      return;
    }
    cobb = Math.round(atr * 2.2);
    activeTab = 'severity';
  }
</script>

<svelte:head>
  <title>{t.title} | Scoliosis self-monitoring</title>
  <meta name="description" content="A bilingual scoliosis self-observation and exercise guide." />
</svelte:head>

<header class="site-header">
  <div class="topbar shell">
    <a class="brand" href="#measure" aria-label={t.title}>
      <span class="brand-mark" aria-hidden="true">✳</span>
      <span>{t.title}</span>
    </a>
    <div class="header-actions">
      <span class="badge">{t.badge}</span>
      <button class="language-button" onclick={() => language = language === 'ko' ? 'en' : 'ko'} aria-label={`Switch language to ${t.language}`}>{t.language}</button>
    </div>
  </div>
  <nav class="tabs shell" aria-label={language === 'ko' ? '주요 메뉴' : 'Main navigation'}>
    {#each Object.entries(t.tabs) as [tab, label] (tab)}
      <button class:active={activeTab === tab} class="tab" onclick={() => activeTab = tab} aria-current={activeTab === tab ? 'page' : undefined}>
        <span class="tab-icon" aria-hidden="true">{tab === 'measure' ? '⌁' : tab === 'severity' ? '◉' : '▶'}</span>{label}
      </button>
    {/each}
  </nav>
</header>

<main class="shell main-content">
  {#if activeTab === 'measure'}
    <section class="panel">
      <div class="section-heading">
        <span class="eyebrow">01 / PHOTO MEASUREMENT</span>
        <h1>{t.measureTitle}</h1>
        <p>{t.measureHelp}</p>
      </div>
      <div class="canvas-frame">
        {#if image}
          <canvas bind:this={canvas} width={imageSize.width} height={imageSize.height} onclick={clickCanvas} role="button" tabindex="0" aria-label={language === 'ko' ? '사진에서 두 측정 지점을 선택하세요' : 'Select two measurement points on the photo'}></canvas>
          <button class="reset-button" onclick={resetPoints}>{t.reset}</button>
        {:else}
          <div class="upload-content">
            <span class="upload-icon" aria-hidden="true">↑</span>
            <p>{t.upload}</p>
            <label class="button button-primary">{t.choose}<input type="file" accept="image/*" onchange={loadPhoto} /></label>
          </div>
        {/if}
      </div>
      <div class="measurement-card">
        <div class="measurement-label"><span>{t.atr}</span><strong>{atr.toFixed(1)}° <small>({atrStatus})</small></strong></div>
        <div class="meter"><span style={`width: ${Math.min(atr / 25 * 100, 100)}%`}></span></div>
        <button class="button button-green" onclick={analyzeFromPhoto}>{t.analyze}<span aria-hidden="true">→</span></button>
      </div>
    </section>
  {:else if activeTab === 'severity'}
    <section class="panel">
      <div class="section-heading">
        <span class="eyebrow">02 / CURVE ESTIMATE</span>
        <h1>{t.severityTitle}</h1>
      </div>
      <div class="form-grid">
        <label class="field"><span>{t.cobb}</span><input type="number" min="0" max="80" bind:value={cobb} /><small>{t.estimate}</small></label>
        <label class="field"><span>{t.curve}</span><select bind:value={curveDirection}><option value="right">{t.right}</option><option value="left">{t.left}</option></select></label>
      </div>
      <article class="severity-card" class:normal={severity === 'normal'} class:mild={severity === 'mild'} class:advanced={severity === 'advanced'}>
        <span class="severity-label">{severityCopy.label}</span>
        <h2>{cobb}° · {severityCopy.title}</h2>
        <p>{severityCopy.desc}</p>
      </article>
      <p class="medical-note"><span aria-hidden="true">ⓘ</span> {t.warning}</p>
    </section>
  {:else}
    <section class="panel exercise-panel">
      <div class="section-heading">
        <span class="eyebrow">03 / MOVEMENT GUIDE</span>
        <h1>{t.exerciseTitle}</h1>
        <p>{t.exerciseIntro}</p>
      </div>
      <div class="exercise-list">
        {#each t.steps as step, index (step.video)}
          <article class="exercise-card">
            <div class="exercise-top"><span class="step-number">0{index + 1}</span><h2>{step.title}</h2><span class="phase">{step.phase}</span></div>
            <div class="exercise-body">
              <div class="video-wrap"><iframe src={step.video} title={step.videoTitle} allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>
              <div class="exercise-copy">
                <p><b>{language === 'ko' ? '수행 방법:' : 'How to:'}</b> {exerciseText(step.how)}</p>
                <p><b>{language === 'ko' ? '핵심 감각:' : 'Focus:'}</b> {exerciseText(step.focus)}</p>
                {#if step.note}<p class="exercise-note"><b>{language === 'ko' ? '주의:' : 'Note:'}</b> {step.note}</p>{/if}
              </div>
            </div>
          </article>
        {/each}
      </div>
    </section>
  {/if}
</main>

{#if toast}<div class="toast" role="status">{toast}</div>{/if}
<footer class="shell footer">SpineCare BioLab <span>·</span> {language === 'ko' ? '자가 관찰 도구' : 'Self-observation guide'}</footer>
