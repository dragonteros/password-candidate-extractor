<script>
  const table2 = {
    choseong: 'r R s e E f a q Q t T d w W c z x v g'.split(' '),
    jungseong: 'k o i O j p u P h hk ho hl y n nj np nl b m ml l'.split(' '),
    jongseong: ' r R rt s sw sg e f fr fa fq ft fx fv fg a q qt t T d w c z x v g'.split(
      ' '
    ),
    from: '`1234567890-=\\[];\',./~!@#$%^&*()_+|{}:"<>?',
    to: '`1234567890-=\\[];\',./~!@#$%^&*()_+|{}:"<>?',
    absent: /[^!-@[-`{-~ㄱ-ㅎㅏ-ㅣ가-힣]+/g,
  }
  const table390 = {
    choseong: "k kk h u uu y i ; ;; n nn j l ll o 0 ' p m".split(' '),
    jungseong: 'f r 6 R t c e 7 v /f /r /d 4 b 9t 9c 9d 5 g 8 d'.split(' '),
    jongseong: ' x F xq s s! S A w D C w3 wq wW wQ V z 3 X q 2 a ! Z E W Q 1'.split(
      ' '
    ),
    from: '`1234567890-=\\[];\',./~!@#$%^&*()_+|{}:"<>?',
    to: '`M<>JKLUION-=\\[]TH,.G~B@#$%^&*()_+|{}:"YP?',
    absent: /[^!-@[-`{-~ㄱ-ㅎㅏ-ㅣ가-힣]+/g,
  }
  const table391 = {
    choseong: "k kk h u uu y i ; ;; n nn j l ll o 0 ' p m".split(' '),
    jungseong: 'f r 6 G t c e 7 v /f /r /d 4 b 9t 9c 9d 5 g 8 d'.split(' '),
    jongseong: ' x ! V s E S A w @ F D T % $ R z 3 X q 2 a # Z C W Q 1'.split(
      ' '
    ),
    from: '?-",.!0123456789·:\\*※=“”\'~()%;<>/+',
    to: 'BNM,.?HJKL:YUIOP"\\|`~^&*()[-{_]=}+',
    absent: /[^!"%'-?\\~ㄱ-ㅎㅏ-ㅣ가-힣·“”※]+/g,
  }

  let layout = '390'
  $: table = layout === '2' ? table2 : layout === '390' ? table390 : table391

  let absent = 'break'
  let whitespace = 'skip'
  let otherskip = ', \' "'
  let otherbreak = '. ;'

  let pwBehavior = 'except'
  let pwSpecial = '[ ] { }'
  let pwReqNumber = true
  let pwReqUpper = false
  let pwReqSpecial = true
  let minLength = 8
  let maxLength = 20
  let pwCharPattern
  $: {
    let chars = pwSpecial
      .replace(/\s+/g, '')
      .replace(/[.*+?^${}()|[\]\\]/g, '\\$&')
    if (pwBehavior === 'except') {
      pwCharPattern = RegExp('^[^' + chars + ']*$', 'i')
    } else {
      pwCharPattern = new RegExp('^[a-zA-Zd' + chars + ']*$', 'i')
    }
  }

  let value =
    '유구한 역사와 전통에 빛나는 우리 대한국민은 3·1운동으로 건립된 대한민국임시정부의 법통과 불의에 항거한 4·19민주이념을 계승하고, 조국의 민주개혁과 평화적 통일의 사명에 입각하여 정의·인도와 동포애로써 민족의 단결을 공고히 하고, 모든 사회적 폐습과 불의를 타파하며, 자율과 조화를 바탕으로 자유민주적 기본질서를 더욱 확고히 하여 정치·경제·사회·문화의 모든 영역에 있어서 각인의 기회를 균등히 하고, 능력을 최고도로 발휘하게 하며, 자유와 권리에 따르는 책임과 의무를 완수하게 하여, 안으로는 국민생활의 균등한 향상을 기하고 밖으로는 항구적인 세계평화와 인류공영에 이바지함으로써 우리들과 우리들의 자손의 안전과 자유와 행복을 영원히 확보할 것을 다짐하면서 1948년 7월 12일에 제정되고 8차에 걸쳐 개정된 헌법을 이제 국회의 의결을 거쳐 국민투표에 의하여 개정한다.'
  let candidates = []
  function extract() {
    candidates = prepSource(value)
      .map((text) => text.split('').map(summarize))
      .flatMap(getAllowedSubseq)
      .flatMap(getReqSubseq)
  }

  function getAllowedSubseq(sylls) {
    let pieces = []
    let piece = []
    for (let syll of sylls) {
      if (syll.allowed) piece.push(syll)
      else if (piece.length) {
        pieces.push(piece)
        piece = []
      }
    }
    if (piece.length) pieces.push(piece)
    return pieces
  }

  function getRegularSubseq(syllables) {
    // when no pw requirement is given
    const totalLength = syllables.reduce((sum, syll) => sum + syll.length, 0)
    if (totalLength < minLength) return []
    let origs = syllables.map((syll) => syll.orig).join('')
    let cur = 0
    for (var left = 0; left < syllables.length; left++) {
      cur += syllables[left].length
      if (cur > maxLength) break
    }
    cur = 0
    for (var right = syllables.length; right > 0; right--) {
      cur += syllables[right - 1].length
      if (cur > maxLength) break
    }
    if (left <= right) {
      return [
        [
          origs.slice(0, left),
          '',
          origs.slice(right),
          origs.slice(left, right),
        ],
      ]
    } else {
      return [
        [
          origs.slice(0, right),
          origs.slice(right, left),
          origs.slice(left),
          '',
        ],
      ]
    }
  }

  function getReqSubseq(syllables) {
    let conditions = []
    if (pwReqNumber) conditions.push((x) => x.hasNumber)
    if (pwReqUpper) conditions.push((x) => x.hasUpper)
    if (pwReqSpecial) conditions.push((x) => x.hasSpecial)
    if (conditions.length === 0) return getRegularSubseq(syllables)
    let cursors = conditions.map(() => -1)

    let idxMap = []
    let origs = []
    for (let syll of syllables) {
      let item = [idxMap.length, idxMap.length + syll.length]
      for (let i = 0; i < syll.length; i++) idxMap.push(item)

      origs.push(syll.orig)
      for (let i = 1; i < syll.length; i++) origs.push('')
    }
    if (idxMap[idxMap.length - 1][1] < minLength) return []

    let cores = [] // 3-tuple of each part of string
    let cur = 0
    let lastLeft = -1

    for (let i = 0; i < syllables.length; i++) {
      for (let j = 0; j < conditions.length; j++) {
        if (conditions[j](syllables[i])) cursors[j] = cur
      }
      cur += syllables[i].length

      let coreRight = cur
      let coreLeft = Math.min(...cursors)
      if (lastLeft === coreLeft) continue
      lastLeft = coreLeft

      let wingLeft = idxMap[Math.max(0, coreRight - maxLength)][0]
      while (wingLeft <= coreLeft) {
        let right = idxMap[Math.min(wingLeft + minLength, idxMap.length - 1)][1]
        if (right - coreLeft <= maxLength) break
        wingLeft = idxMap[wingLeft][1]
      }
      if (wingLeft > coreLeft) continue

      let wingRight =
        idxMap[Math.min(coreLeft + maxLength, idxMap.length - 1)][1]
      while (wingRight >= coreRight) {
        let left = idxMap[Math.max(0, wingRight - minLength)][0]
        if (wingRight - left <= maxLength) break
        wingRight = idxMap[wingRight - 1][0]
      }
      if (wingRight < coreRight) continue

      cores.push([
        origs.slice(wingLeft, coreLeft).join(''),
        origs.slice(coreLeft, coreRight).join(''),
        origs.slice(coreRight, wingRight).join(''),
        '',
      ])
    }
    return cores
  }

  function buildPattern(chars) {
    chars = chars.replace(/\s+/g, '').replace(/[.*+?^${}()|[\]\\]/g, '\\$&')
    return new RegExp('[' + chars + ']+', 'g')
  }

  function mapPattern(items, pattern, behavior = 'break') {
    if (behavior === 'skip')
      return items.map((item) => item.replace(pattern, ''))
    return items.reduce((acc, x) => acc.concat(x.split(pattern)), [])
  }

  function prepSource(text) {
    let candidates = [text]
    candidates = mapPattern(candidates, /\s+/g, whitespace)
    candidates = mapPattern(candidates, table.absent, absent)
    candidates = mapPattern(candidates, buildPattern(otherskip), 'skip')
    candidates = mapPattern(candidates, buildPattern(otherbreak), 'break')
    return candidates
  }

  const compatibleChoseong = 'ㄱㄲㄴㄷㄸㄹㅁㅂㅃㅅㅆㅇㅈㅉㅊㅋㅌㅍㅎ'
  const compatibleJongseong =
    'ㄱㄲㄳㄴㄵㄶㄷㄹㄺㄻㄼㄽㄾㄿㅀㅁㅂㅄㅅㅆㅇㅈㅊㅋㅌㅍㅎ'
  function decomposeHangeul(char) {
    let code = char.charCodeAt(0)

    if ('O' <= char && char <= 'c') return [null, code - 0x314f, null]
    let idx = compatibleChoseong.indexOf(char)
    if (idx !== -1) return [idx, null, null]
    idx = compatibleJongseong.indexOf(char)
    if (idx !== -1) return [null, null, idx + 1]

    return [
      ((code - 0xac00) / 588) | 0,
      (((code - 0xac00) % 588) / 28) | 0,
      (code - 0xac00) % 28,
    ]
  }

  function convertChar(char) {
    let idx = table.from.indexOf(char)
    if (idx !== -1) return table.to[idx]

    let decomposed = decomposeHangeul(char)
    let converted = ''
    if (decomposed[0] != null) converted += table.choseong[decomposed[0]]
    if (decomposed[1] != null) converted += table.jungseong[decomposed[1]]
    if (decomposed[2] != null) converted += table.jongseong[decomposed[2]]
    return converted
  }

  function summarize(char) {
    const chars = convertChar(char)
    return {
      orig: char,
      chars: chars,
      length: chars.length,
      allowed: pwCharPattern.test(chars),
      hasNumber: /\d/.test(chars),
      hasUpper: /[A-Z]/.test(chars),
      hasSpecial: /[^0-9A-Za-z]/.test(chars),
    }
  }
</script>

<div class="box">
  <main>
    <textarea bind:value />

    <p>
      <button on:click={extract}>추출</button>
    </p>

    {#if candidates.length}
      <ul>
        {#each candidates as candidate}
          <li>
            <span class="leftwing">{candidate[0]}</span><span class="core"
              >{candidate[1]}</span
            ><span>{candidate[3]}</span><span class="rightwing"
              >{candidate[2]}</span
            >
          </li>
        {/each}
      </ul>
    {:else}
      <p>비밀번호 후보가 없습니다.</p>
    {/if}
  </main>

  <nav>
    <h2>설정</h2>

    <h3>원문</h3>
    <h4>사용 자판</h4>
    <label>
      <input type="radio" bind:group={layout} value="2" />
      두벌식
    </label>
    <label>
      <input type="radio" bind:group={layout} value="390" />
      세벌식 390
    </label>
    <label>
      <input type="radio" bind:group={layout} value="391" />
      세벌식 최종
    </label>

    <h4>자판에 있는 글자</h4>
    <label>
      건너뛰기
      <input type="text" bind:value={otherskip} />
    </label>
    <label>
      끊기
      <input type="text" bind:value={otherbreak} />
    </label>

    <h4>자판에 없는 글자</h4>
    <label>
      <input type="radio" bind:group={absent} value="skip" />
      건너뛰기
    </label>
    <label>
      <input type="radio" bind:group={absent} value="break" />
      끊기
    </label>

    <h4>띄어쓰기, 줄 바꿈 등</h4>
    <label>
      <input type="radio" bind:group={whitespace} value="skip" />
      건너뛰기
    </label>
    <label>
      <input type="radio" bind:group={whitespace} value="break" />
      끊기
    </label>

    <h3>비밀번호</h3>
    <h4>비밀번호 길이</h4>
    <label>
      최소
      <input type="number" bind:value={minLength} />
    </label>
    <label>
      최대
      <input type="number" bind:value={maxLength} />
    </label>

    <h4>비밀번호 조건</h4>
    <label>
      <input type="checkbox" bind:checked={pwReqNumber} />
      숫자 포함
    </label>
    <label>
      <input type="checkbox" bind:checked={pwReqUpper} />
      대문자 포함
    </label>
    <label>
      <input type="checkbox" bind:checked={pwReqSpecial} />
      특수문자 포함
    </label>

    <h4>특수문자 조건</h4>
    <input type="text" bind:value={pwSpecial} />만
    <select bind:value={pwBehavior}>
      <option value="only">포함</option>
      <option value="except">제외</option>
    </select>
  </nav>
</div>

<style>
  .leftwing {
    background-color: rgb(213, 218, 255);
  }
  .rightwing {
    background-color: rgb(255, 208, 217);
  }
  .core {
    background-color: rgb(234, 189, 255);
  }

  @media (min-width: 550px) {
    .box {
      display: flex;
    }

    main {
      flex: 1;
      margin: 10px;
    }

    nav {
      flex: 0 0 310px;
    }
  }

  nav {
    padding: 0 10px;
    background-color: rgb(251, 244, 255);
  }

  textarea {
    width: 100%;
    height: 300px;
  }

  button {
    width: 100%;
  }
</style>
