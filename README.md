# Smooth Scroll Implementation

This repository contains three different implementations for smooth scrolling: GSAP ScrollSmoother, Lenis, and Locomotive Scroll. Each of these can be used in a **React** or **Next.js** project. This guide will walk you through the initialization steps for each smooth scrolling library.

## Requirements

This code can be used in your **React** or **Next.js** project. Before you start, ensure that you have one of these frameworks set up.

### Libraries:

- **[React](https://reactjs.org/)** (or **[Next.js](https://nextjs.org/)**)
- **[GSAP](https://greensock.com/gsap/)** (for GSAP ScrollSmoother and ScrollTrigger)
- **[Lenis](https://www.studiofreight.com/lenis/)** (for smooth scrolling using Lenis)
- **[Locomotive Scroll](https://locomotivemtl.github.io/locomotive-scroll/)** (for smooth scrolling with Locomotive)


## Installation

First, install the required packages for each smooth scroll implementation:

```bash
# GSAP ScrollSmoother & ScrollTrigger
npm install gsap

# Lenis
npm install @studio-freight/lenis

# Locomotive Scroll
npm install locomotive-scroll
```

# GSAP ScrollSmoother Setup

```easycode
import { useRef, useEffect, useContext } from "react";
import { gsap } from "gsap";
import { ScrollSmoother } from "gsap/ScrollSmoother";
import { ScrollTrigger } from "gsap/ScrollTrigger";
import { ScrollToPlugin } from "gsap/ScrollToPlugin";


if (typeof window !== "undefined") {
    gsap.registerPlugin(ScrollTrigger, ScrollSmoother, ScrollToPlugin);
}

export const useSmoothScroll = (isMobile) => {
    const el = useRef(null);
    const { completed } = useContext(TransitionContext);

    useEffect(() => {
        let smoother;

        if (typeof window !== "undefined" && !ScrollTrigger.isTouch) {
            smoother = ScrollSmoother.create({
                smooth: !isMobile ? 3 : 2, // Smoothness level based on device
                effects: !isMobile, // Enable GSAP effects on non-mobile
                smoothTouch: 0.1,
                ignoreMobileResize: true,
                content: "#smooth-content",
                wrapper: "#smooth-wrapper",
            });

            el.current = smoother;
        }

        ScrollTrigger.refresh();

        return () => {
            if (smoother) {
                smoother.kill(); 
                el.current = null;
            }
            ScrollTrigger.clearMatchMedia();
            ScrollTrigger.getAll().forEach((trigger) => trigger.kill());
        };
    }, [isMobile]);

    useEffect(() => {
        if (completed && el.current) {
            el.current.refresh();
        }
    }, [completed]);

    return el;
};

```
# Lenis Setup

```easycode
import { useEffect, useRef } from 'react';
import Lenis from '@studio-freight/lenis';
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);

export const useLenis = () => {
    const lenisRef = useRef(null);

    useEffect(() => {
        const lenis = new Lenis({
            duration: 1.2,
            easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t)), // Smooth easing function
            direction: 'vertical',
            smooth: true,
            mouseMultiplier: 1,
            lerp: 0.1,
            smoothTouch: false,
            touchMultiplier: 2,
            infinite: false,
            wrapper: window,
            content: document.body,
        });

        lenisRef.current = lenis;

        lenis.on('scroll', ({ scroll, limit, velocity, direction, progress }) => {
            ScrollTrigger.update();
        });

        gsap.ticker.add((time) => {
            lenis.raf(time * 1000);
        });

        gsap.ticker.lagSmoothing(0);

        function raf(time) {
            lenis.raf(time);
            requestAnimationFrame(raf);
        }

        requestAnimationFrame(raf);

        return () => lenis.destroy();
    }, []);

    return lenisRef;
};

```

# Locomotive Scroll Setup

```easycode
import { useEffect, useRef } from "react";
import gsap from "gsap";
import ScrollTrigger from "gsap/dist/ScrollTrigger";
import "locomotive-scroll/dist/locomotive-scroll.css";
import dynamic from "next/dynamic";

const LocomotiveScroll = dynamic(() => import("locomotive-scroll").then(mod => mod.default), { ssr: false });

gsap.registerPlugin(ScrollTrigger);

const useLocomotiveScroll = ({ pathname }) => {
    const containerRef = useRef(null);
    const scrollInstanceRef = useRef(null);

    useEffect(() => {
        const initLocomotiveScroll = async () => {
            if (!containerRef.current || typeof window === "undefined") return;

            const LocomotiveScrollClass = await import("locomotive-scroll").then((mod) => mod.default);

            scrollInstanceRef.current = new LocomotiveScrollClass({
                el: containerRef.current,
                smooth: true,
                smartphone: {
                    smooth: false,
                    breakpoint: 0,
                    getDirection: true,
                },
                tablet: {
                    smooth: true,
                    inertia: 0.8,
                    breakpoint: 0,
                    getDirection: true,
                },
                getDirection: true,
            });

            scrollInstanceRef.current.on('scroll', ScrollTrigger.update);

            ScrollTrigger.scrollerProxy(containerRef.current, {
                scrollTop(value) {
                    return arguments.length
                        ? scrollInstanceRef.current.scrollTo(value, { duration: 0, disableLerp: true })
                        : scrollInstanceRef.current.scroll.instance.scroll.y;
                },
                getBoundingClientRect() {
                    return {
                        top: 0,
                        left: 0,
                        width: window.innerWidth,
                        height: window.innerHeight,
                    };
                },
                pinType: containerRef.current.style.transform ? "transform" : "fixed",
            });

            ScrollTrigger.addEventListener("refresh", () => scrollInstanceRef.current.update());
            ScrollTrigger.refresh();
        };

        initLocomotiveScroll();

        return () => {
            if (scrollInstanceRef.current) {
                scrollInstanceRef.current.destroy();
                ScrollTrigger.removeEventListener("refresh", () => scrollInstanceRef.current.update());
            }
        };
    }, [pathname]);

    return { containerRef, scrollInstance: scrollInstanceRef };
};

export default useLocomotiveScroll;

```

# Conclusion

You can choose any of the three smooth scroll libraries — **[GSAP ScrollSmoother](https://gsap.com/docs/v3/Plugins/ScrollSmoother/)**, **[Lenis](https://lenis.darkroom.engineering/)**, or **[Locomotive Scroll](https://github.com/locomotivemtl/locomotive-scroll)** — based on your project requirements. These libraries offer different configurations and performance optimizations, so you can choose the one that fits your project needs.
