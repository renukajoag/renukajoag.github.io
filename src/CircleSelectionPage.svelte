<script>
  export let state;
  export let expandedIndex;
  export let expandOrigin;
  export let sections;
export let expandedRenderKey;
  export let hoveredIndex;
export let playExpandAnimation = false;
  export let circleGoingToCorner = false;
  export let circleComingFromCorner = false;

  export let handleCircleClick;
  export let handleCornerPillClick;

  export let originClass;
export let returningFromDeep = false;

export let enteringCircles = false;
  $: isCirclesVisible =
    ['circles', 'expanded', 'deep', 'collapsing'].includes(state)
    || circleGoingToCorner
    || circleComingFromCorner;

  $: showBigCircle =
    (state === 'expanded' || circleGoingToCorner)
    && expandedIndex !== null;
</script>

<div
  class="circles-page"
  class:circles-page-visible={isCirclesVisible}
>

  <!-- Corner slots -->
  {#if (state === 'expanded' || state === 'deep' || circleGoingToCorner || circleComingFromCorner) && expandedIndex !== null}
    <div class="corner-pill-slots">
      {#each [0,1,2] as slotIdx}

        {#if slotIdx === expandedIndex
          && expandedIndex !== 0
          && (state === 'deep' || circleGoingToCorner || circleComingFromCorner)}

          <div
            class="corner-pill-slot"
            on:click={() => state === 'deep' && handleCornerPillClick(slotIdx)}
          >
            <div
              class="corner-circle
              {circleGoingToCorner
                ? 'corner-shrink-0'
                : circleComingFromCorner
                  ? 'corner-grow-0'
                  : ''}"
            ></div>
          </div>

        {:else if slotIdx !== expandedIndex}

          <div
            class="corner-pill-slot"
            on:click={() => handleCornerPillClick(slotIdx)}
            title={sections[slotIdx].label}
          >
            <div class="corner-circle corner-shrink-{slotIdx}"></div>
          </div>

        {:else}

          <div class="corner-pill-slot slot-empty"></div>

        {/if}

      {/each}
    </div>
  {/if}

  <!-- Expanded circle -->
  {#if showBigCircle}
    {#key expandedRenderKey}
      <div class="expanded-scene" class:circle-exit={circleGoingToCorner}>
        <div
          class="big-expanded-circle
{
            playExpandAnimation
      ? originClass(expandOrigin)
      : ''
  }
  {circleGoingToCorner ? 'circle-to-corner' : ''}"
          style="background:{sections[expandedIndex].color}"
        >

          {#if !circleGoingToCorner}

            <div class="expanded-items">
              {#each (
                sections[expandedIndex].hasDeepSection
                  ? [
                      'Who I am',
                      'What makes me, me',
                      'What I care about',
                      'Outside of design',
                      'Right now'
                    ]
                  : [
                      'Brand identity',
                      'Editorial design',
                      'Illustration',
                      'Motion',
                      'Packaging'
                    ]
              ) as item, idx}

                <span
                  class="expanded-item"
                  style="animation-delay:{0.25 + idx * 0.1}s"
                >
                  {item}
                </span>

              {/each}
            </div>

            {#if sections[expandedIndex].hasDeepSection}
              <div class="scroll-hint">
                scroll to explore ↓
              </div>
            {/if}

          {/if}

        </div>
      </div>
    {/key}
  {/if}

  <!-- Three circles row -->
{#if state === 'circles' && !enteringCircles}
    <div class="circles-row">

      {#each sections as s, i}

        <div class="circle-item">

          <button
            class="big-circle"
            style="background:{hoveredIndex === i ? '#8FAF9A' : s.color}"
            on:mouseenter={() => hoveredIndex = i}
            on:mouseleave={() => hoveredIndex = null}
            on:click={() => handleCircleClick(i)}
            aria-label={s.label}
          ></button>

          <span class="circle-label">
            {s.label}
          </span>

        </div>

      {/each}

    </div>
  {/if}

</div>

<style>
.circles-page{
  position:absolute;
  inset:0;
  z-index:2;

  display:flex;
  align-items:center;
  justify-content:center;

    pointer-events:none;
}

.circles-page.circles-page-visible{
  pointer-events:all;
}

  .corner-pill-slots{
    position:absolute;
    top:4.5vh;
    left:4vw;
    display:flex;
    gap:0.7vw;
    z-index:12;
    align-items:center;
  }

  .corner-pill-slot{
    width:clamp(18px,2.6vw,38px);
    height:clamp(18px,2.6vw,38px);
    display:flex;
    align-items:center;
    justify-content:center;
    cursor:pointer;
  }

  .slot-empty{
    pointer-events:none;
  }

  .corner-circle{
    width:100%;
    height:100%;
    border-radius:50%;
    background:#E35D5B;
    transition:transform 0.15s ease;
  }

  .corner-pill-slot:not(.slot-empty):hover .corner-circle{
    transform:scale(1.2);
  }

  .corner-shrink-0{
    animation:shrink0 0.65s cubic-bezier(0.4,0,0.2,1) forwards;
  }

  .corner-shrink-1{
    animation:shrink1 0.65s cubic-bezier(0.4,0,0.2,1) forwards;
  }

  .corner-shrink-2{
    animation:shrink2 0.65s cubic-bezier(0.4,0,0.2,1) forwards;
  }

  .corner-grow-0{
    animation:shrink0 0.65s cubic-bezier(0.4,0,0.2,1) reverse forwards;
  }

  @keyframes shrink0{
    from{
      transform:
        translate(calc(50vw - 4vw - 1.3vw),
        calc(50vh - 4.5vh - 1.3vw))
        scale(18)
    }

    to{
      transform:translate(0,0) scale(1)
    }
  }

  @keyframes shrink1{
    from{
      transform:
        translate(calc(50vw - 4vw - 3.3vw - 1.3vw),
        calc(50vh - 4.5vh - 1.3vw))
        scale(18)
    }

    to{
      transform:translate(0,0) scale(1)
    }
  }

  @keyframes shrink2{
    from{
      transform:
        translate(calc(50vw - 4vw - 6.6vw - 1.3vw),
        calc(50vh - 4.5vh - 1.3vw))
        scale(18)
    }

    to{
      transform:translate(0,0) scale(1)
    }
  }

  .circles-row{
    display:flex;
    align-items:center;
    justify-content:center;
    gap:4vw;
  }

  .circle-item{
    display:flex;
    flex-direction:column;
    align-items:center;
    gap:2.5vh;
  }

  .big-circle{
    width:clamp(160px,22vw,340px);
    height:clamp(160px,22vw,340px);
    border-radius:50%;
    border:none;
    cursor:pointer;
    transition:
      background 0.3s ease,
      transform 0.3s cubic-bezier(0.34,1.56,0.64,1);
    display:block;
  }

  .big-circle:hover{
    transform:scale(1.05);
  }

  .circle-label{
    font-family:'Trispace',monospace;
    font-stretch:112.5%;
    font-weight:300;
    font-size:clamp(11px,1.1vw,18px);
    color:#3A2F2B;
    letter-spacing:0.12em;
  }

  .expanded-scene{
    position:absolute;
    inset:0;
    z-index:10;
    display:flex;
    align-items:center;
    justify-content:center;
    pointer-events:none;
  }

  .big-expanded-circle{
    width:clamp(400px,60vmin,720px);
    height:clamp(400px,60vmin,720px);
    border-radius:50%;
    display:flex;
    flex-direction:column;
    align-items:center;
    justify-content:center;
    pointer-events:auto;
  }

  .big-expanded-circle.from-left{
    animation:expandFromLeft 0.75s cubic-bezier(0.16,1,0.3,1) forwards;
  }

  .big-expanded-circle.from-center{
    animation:expandFromCenter 0.75s cubic-bezier(0.16,1,0.3,1) forwards;
  }

  .big-expanded-circle.from-right{
    animation:expandFromRight 0.75s cubic-bezier(0.16,1,0.3,1) forwards;
  }

  .big-expanded-circle.circle-to-corner{
    animation:
      circleToCorner 0.65s cubic-bezier(0.4,0,0.2,1) forwards !important;
  }

  .expanded-items{
    display:flex;
    flex-direction:column;
    align-items:center;
    gap:1.4vh;
  }

  .expanded-item{
    font-family:'Trispace',monospace;
    font-stretch:112.5%;
    font-weight:300;
    font-size:clamp(13px,1.3vw,22px);
    color:#f2eded;
    letter-spacing:0.1em;
    opacity:0;
    animation:itemFadeIn 0.45s ease forwards;
  }

  .scroll-hint{
    font-family:'Trispace',monospace;
    font-stretch:112.5%;
    font-weight:300;
    font-size:clamp(9px,0.75vw,13px);
    color:#f2eded;
    letter-spacing:0.15em;
    opacity:0;
    margin-top:2.5vh;
    animation:itemFadeIn 0.5s ease 1s forwards;
  }

  @keyframes expandFromLeft{
    from{
      opacity:0.4;
      transform:translate(-28vw,0) scale(0.22)
    }

    to{
      opacity:1;
      transform:translate(0,0) scale(1)
    }
  }

  @keyframes expandFromCenter{
    from{
      opacity:0.4;
      transform:scale(0.22)
    }

    to{
      opacity:1;
      transform:scale(1)
    }
  }

  @keyframes expandFromRight{
    from{
      opacity:0.4;
      transform:translate(28vw,0) scale(0.22)
    }

    to{
      opacity:1;
      transform:translate(0,0) scale(1)
    }
  }

  @keyframes itemFadeIn{
    from{
      opacity:0;
      transform:translateY(10px)
    }

    to{
      opacity:1;
      transform:translateY(0)
    }
  }
</style>
