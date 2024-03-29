<template lang='pug'>
div.sui-steps(:class="{ line: type === 'line', ring: type === 'ring' }" :style="{'--ring-count': steps.length, '--ring-status': complete + (type === 'ring' ? 1 : 0)}")
    svg
        circle.step(r="1.5em" cx="50%" cy="50%")
    svg
        circle.step(r="1.5em" cx="50%" cy="50%")
    sui-step(v-for="(step,idx) in steps" :step-size="steps.length - 1" :type="type" :steps="steps" :idx="idx" :complete="complete")
</template>

<script>
export default {
    name: 'sui-steps',
    props: {
        type: String,
        steps: Array,
        completed: {
            type: Number,
            default: 0
        }
    },
    computed: {
        complete() {
            return this.steps.length < this.completed ? this.steps.length - 1 : this.completed > this.steps.length - 1 ? this.steps.length - 1 : this.completed;
        }
    }
};
</script>
<style scoped lang="less">
@import '../assets/viewport.less';

div.sui-steps {
    display: flex;
    justify-content: space-between;

    &.line {
        overflow: hidden;

        svg {
            display: none;
        }

        .step {
            &.active,
            &.visible {
                display: inline-flex;
            }
        }
    }

    div.step {
        display: none;
        width: 100%;
        cursor: default;
        padding-right: var(--padding);
        padding-top: calc(var(--padding) / 4);
        color: var(--content-text_placeholder, #b3b3b3);
        margin-right: calc(var(--padding) / 4);
        border-top: 3px solid var(--content-text_placeholder, #b3b3b3);;
        line-height: 1.2;

        & > .left {
            padding-right: .5em;

            i {
                vertical-align: middle;
            }
        }

        &.complete {
            opacity: .5;
            border-top: 3px solid var(--saturate, #4848db);
            color: var(--button-nude, inherit);
        }

        &.current {
            font-weight: normal;
            border-top: 3px solid var(--saturate, #4848db);
            color: var(--button-nude, inherit);
        }
    }

    &.ring {
        counter-reset: --ring-status var(--ring-status);
        position: relative;

        & div.step.current {
            flex-grow: 1;
            border-top: none;
            padding-top: 0;
            font-size: 1.333rem;
        }

        &::before {
            content: counter(--ring-status);
            position: absolute;
            display: flex;
            height: calc(3em + .5em);
            width: calc(3em + .5em);
            text-align: center;
            justify-content: center;
            align-items: center;
            font-weight: normal;
            color: var(--button-nude, inherit);
        }

        padding-bottom: 0;

        & > svg {
            flex-shrink: 0;

            --PI: 3.14159265358979;
            --circum: calc(3 * var(--PI));

            margin-right: var(--padding);
            height: calc(3em + .5em);
            width: calc(3em + .5em);
            --offset: calc(var(--circum) - ((100 / var(--ring-count)) * var(--ring-status)) / 100 * var(--circum));

            circle {
                stroke-width: .5em;
                fill: transparent;
            }

            &:first-child {
                display: inline-block;
                position: absolute;

                circle {
                    stroke: var(--saturate_transparent, rgba(72, 72, 219, 0.22));
                }

                & + svg {
                    display: block;
                    transform: rotate(-90deg);
                    transform-origin: 50% 50%;

                    circle {
                        stroke: var(--saturate, #4848db);
                        stroke-dasharray: calc(var(--circum) * 1em);
                        stroke-dashoffset: calc(var(--offset) * 1em);
                        transition: stroke-dashoffset .25s;
                    }
                }
            }
        }

        div.step.current {
            width: unset;
            display: inline-flex;
        }
    }
}
</style>
