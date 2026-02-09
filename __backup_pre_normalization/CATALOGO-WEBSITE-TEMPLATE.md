# CATALOGO-WEBSITE-TEMPLATE

ESPANSIONE CATALOGO-WEBSITE-TEMPLATE NEXT.JS 14
§ LANDING PAGE SECTIONS
Hero Section Variants
tsx
// components/hero/centered.tsx
export function HeroCentered() {
  return (
    <section className="relative overflow-hidden bg-gradient-to-b from-white to-gray-50 py-24 sm:py-32">
      <div className="mx-auto max-w-7xl px-6 lg:px-8">
        <div className="mx-auto max-w-2xl text-center">
          <h1 className="text-4xl font-bold tracking-tight text-gray-900 sm:text-6xl">
            Build your next idea faster
          </h1>
          <p className="mt-6 text-lg leading-8 text-gray-600">
            Beautifully designed, fully responsive components and templates for
            your Next.js projects. Open source and production ready.
          </p>
          <div className="mt-10 flex items-center justify-center gap-x-6">
            <a
              href="#"
              className="rounded-md bg-indigo-600 px-3.5 py-2.5 text-sm font-semibold text-white shadow-sm hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600"
            >
              Get started
            </a>
            <a
              href="#"
              className="text-sm font-semibold leading-6 text-gray-900"
            >
              Learn more <span aria-hidden="true">→</span>
            </a>
          </div>
        </div>
        <div className="mt-16 flow-root sm:mt-24">
          <div className="-m-2 rounded-xl bg-gray-900/5 p-2 ring-1 ring-inset ring-gray-900/10 lg:-m-4 lg:rounded-2xl lg:p-4">
            <img
              src="https://tailwindui.com/img/component-images/project-app-screenshot.png"
              alt="App screenshot"
              className="rounded-md shadow-2xl ring-1 ring-gray-900/10"
            />
          </div>
        </div>
      </div>
    </section>
  );
}
tsx
// components/hero/split.tsx
export function HeroSplit() {
  return (
    <div className="relative bg-white">
      <div className="mx-auto max-w-7xl lg:grid lg:grid-cols-12 lg:gap-x-8 lg:px-8">
        <div className="px-6 pb-24 pt-10 sm:pb-32 lg:col-span-7 lg:px-0 lg:pb-56 lg:pt-48 xl:col-span-6">
          <div className="mx-auto max-w-2xl lg:mx-0">
            <h1 className="mt-24 text-4xl font-bold tracking-tight text-gray-900 sm:mt-10 sm:text-6xl">
              Data to enrich your online business
            </h1>
            <p className="mt-6 text-lg leading-8 text-gray-600">
              Anim aute id magna aliqua ad ad non deserunt sunt. Qui irure qui
              lorem cupidatat commodo. Elit sunt amet fugiat veniam occaecat
              fugiat aliqua.
            </p>
            <div className="mt-10 flex items-center gap-x-6">
              <a
                href="#"
                className="rounded-md bg-indigo-600 px-3.5 py-2.5 text-sm font-semibold text-white shadow-sm hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600"
              >
                Get started
              </a>
              <a
                href="#"
                className="text-sm font-semibold leading-6 text-gray-900"
              >
                Learn more <span aria-hidden="true">→</span>
              </a>
            </div>
          </div>
        </div>
        <div className="relative lg:col-span-5 lg:-mr-8 xl:absolute xl:inset-0 xl:left-1/2 xl:mr-0">
          <img
            className="aspect-[3/2] w-full bg-gray-50 object-cover lg:absolute lg:inset-0 lg:aspect-auto lg:h-full"
            src="https://images.unsplash.com/photo-1498758536662-35b82cd15e29?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=2102&q=80"
            alt=""
          />
        </div>
      </div>
    </div>
  );
}
Features Section
tsx
// components/features/grid.tsx
export function FeaturesGrid() {
  const features = [
    {
      name: "Push to deploy",
      description:
        "Morbi viverra dui mi arcu sed. Tellus semper adipiscing suspendisse semper morbi.",
      icon: CloudArrowUpIcon,
    },
    {
      name: "SSL certificates",
      description:
        "Sit quis amet rutrum tellus ullamcorper ultricies libero dolor eget.",
      icon: LockClosedIcon,
    },
    {
      name: "Simple queues",
      description:
        "Quisque est vel vulputate cursus. Risus proin diam nunc commodo.",
      icon: ArrowPathIcon,
    },
    {
      name: "Advanced security",
      description:
        "Arcu egestas dolor vel iaculis in ipsum mauris. Tincidunt mattis aliquet.",
      icon: FingerPrintIcon,
    },
  ];

  return (
    <div className="bg-white py-24 sm:py-32">
      <div className="mx-auto max-w-7xl px-6 lg:px-8">
        <div className="mx-auto max-w-2xl lg:text-center">
          <h2 className="text-base font-semibold leading-7 text-indigo-600">
            Deploy faster
          </h2>
          <p className="mt-2 text-3xl font-bold tracking-tight text-gray-900 sm:text-4xl">
            Everything you need to deploy your app
          </p>
          <p className="mt-6 text-lg leading-8 text-gray-600">
            Quis tellus eget adipiscing convallis sit sit eget aliquet quis.
            Suspendisse eget egestas a elementum pulvinar et feugiat blandit at.
            In mi viverra elit nunc.
          </p>
        </div>
        <div className="mx-auto mt-16 max-w-2xl sm:mt-20 lg:mt-24 lg:max-w-none">
          <dl className="grid max-w-xl grid-cols-1 gap-x-8 gap-y-16 lg:max-w-none lg:grid-cols-4">
            {features.map((feature) => (
              <div key={feature.name} className="flex flex-col">
                <dt className="flex items-center gap-x-3 text-base font-semibold leading-7 text-gray-900">
                  <feature.icon
                    className="h-5 w-5 flex-none text-indigo-600"
                    aria-hidden="true"
                  />
                  {feature.name}
                </dt>
                <dd className="mt-4 flex flex-auto flex-col text-base leading-7 text-gray-600">
                  <p className="flex-auto">{feature.description}</p>
                </dd>
              </div>
            ))}
          </dl>
        </div>
      </div>
    </div>
  );
}
Testimonials Carousel
tsx
// components/testimonials/carousel.tsx
'use client';

import { useState, useEffect } from 'react';
import { ChevronLeftIcon, ChevronRightIcon } from '@heroicons/react/20/solid';

const testimonials = [
  {
    id: 1,
    content:
      "This template has saved us thousands of hours of work. We're able to spin up projects in hours, not weeks.",
    author: {
      name: 'Judith Black',
      role: 'CEO of Workcation',
      imageUrl:
        'https://images.unsplash.com/photo-1494790108755-2616b612b786?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=facearea&facepad=2&w=256&h=256&q=80',
    },
  },
  {
    id: 2,
    content:
      "The quality of components is outstanding. Our development velocity increased by 40% after adoption.",
    author: {
      name: 'Michael Foster',
      role: 'CTO at Syntax',
      imageUrl:
        'https://images.unsplash.com/photo-1519244703995-f4e0f30006d5?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=facearea&facepad=2&w=256&h=256&q=80',
    },
  },
  {
    id: 3,
    content:
      "We've reduced our time-to-market by 60% using these templates. The responsive patterns are brilliant.",
    author: {
      name: 'Lindsay Walton',
      role: 'Front-end Developer',
      imageUrl:
        'https://images.unsplash.com/photo-1517841905240-472988babdf9?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=facearea&facepad=2&w=256&h=256&q=80',
    },
  },
];

export function TestimonialsCarousel() {
  const [currentIndex, setCurrentIndex] = useState(0);

  const next = () => {
    setCurrentIndex((prevIndex) => 
      prevIndex === testimonials.length - 1 ? 0 : prevIndex + 1
    );
  };

  const prev = () => {
    setCurrentIndex((prevIndex) => 
      prevIndex === 0 ? testimonials.length - 1 : prevIndex - 1
    );
  };

  useEffect(() => {
    const interval = setInterval(next, 5000);
    return () => clearInterval(interval);
  }, []);

  return (
    <div className="bg-white py-24 sm:py-32">
      <div className="mx-auto max-w-7xl px-6 lg:px-8">
        <div className="mx-auto max-w-2xl text-center">
          <h2 className="text-3xl font-bold tracking-tight text-gray-900 sm:text-4xl">
            Trusted by developers worldwide
          </h2>
        </div>
        <div className="relative mx-auto mt-16 max-w-2xl">
          <div className="overflow-hidden">
            <div
              className="flex transition-transform duration-300 ease-in-out"
              style={{ transform: `translateX(-${currentIndex * 100}%)` }}
            >
              {testimonials.map((testimonial) => (
                <div
                  key={testimonial.id}
                  className="w-full flex-shrink-0 px-4"
                >
                  <figure className="rounded-2xl bg-gray-50 p-8 text-sm leading-6">
                    <blockquote className="text-gray-900">
                      <p>{testimonial.content}</p>
                    </blockquote>
                    <figcaption className="mt-6 flex items-center gap-x-4">
                      <img
                        className="h-10 w-10 rounded-full bg-gray-50"
                        src={testimonial.author.imageUrl}
                        alt=""
                      />
                      <div>
                        <div className="font-semibold text-gray-900">
                          {testimonial.author.name}
                        </div>
                        <div className="text-gray-600">
                          {testimonial.author.role}
                        </div>
                      </div>
                    </figcaption>
                  </figure>
                </div>
              ))}
            </div>
          </div>
          <button
            onClick={prev}
            className="absolute left-0 top-1/2 -translate-y-1/2 rounded-full bg-white p-2 shadow-lg"
          >
            <ChevronLeftIcon className="h-5 w-5 text-gray-700" />
          </button>
          <button
            onClick={next}
            className="absolute right-0 top-1/2 -translate-y-1/2 rounded-full bg-white p-2 shadow-lg"
          >
            <ChevronRightIcon className="h-5 w-5 text-gray-700" />
          </button>
          <div className="mt-6 flex justify-center gap-2">
            {testimonials.map((_, index) => (
              <button
                key={index}
                onClick={() => setCurrentIndex(index)}
                className={`h-2 w-2 rounded-full ${
                  index === currentIndex ? 'bg-indigo-600' : 'bg-gray-300'
                }`}
              />
            ))}
          </div>
        </div>
      </div>
    </div>
  );
}
Pricing Section
tsx
// components/pricing/cards.tsx
export function PricingCards() {
  const plans = [
    {
      name: 'Basic',
      price: '$29',
      description: 'Perfect for small projects',
      features: [
        'Up to 5 projects',
        'Basic components',
        'Email support',
        '1GB storage',
        'Community access',
      ],
      featured: false,
    },
    {
      name: 'Pro',
      price: '$79',
      description: 'For growing businesses',
      features: [
        'Up to 20 projects',
        'Advanced components',
        'Priority support',
        '10GB storage',
        'Analytics dashboard',
        'Custom branding',
      ],
      featured: true,
    },
    {
      name: 'Enterprise',
      price: '$199',
      description: 'For large organizations',
      features: [
        'Unlimited projects',
        'All components',
        '24/7 phone support',
        '100GB storage',
        'Advanced analytics',
        'Custom development',
        'SLA guarantee',
      ],
      featured: false,
    },
  ];

  return (
    <div className="bg-white py-24 sm:py-32">
      <div className="mx-auto max-w-7xl px-6 lg:px-8">
        <div className="mx-auto max-w-4xl text-center">
          <h2 className="text-base font-semibold leading-7 text-indigo-600">
            Pricing
          </h2>
          <p className="mt-2 text-4xl font-bold tracking-tight text-gray-900 sm:text-5xl">
            Choose the right plan for you
          </p>
        </div>
        <p className="mx-auto mt-6 max-w-2xl text-center text-lg leading-8 text-gray-600">
          Distinctio et nulla eum soluta et neque labore quibusdam. Saepe et
          quasi iusto modi velit ut non voluptas.
        </p>
        <div className="isolate mx-auto mt-16 grid max-w-md grid-cols-1 gap-y-8 sm:mt-20 lg:mx-0 lg:max-w-none lg:grid-cols-3">
          {plans.map((plan, planIdx) => (
            <div
              key={plan.name}
              className={`
                flex flex-col justify-between rounded-3xl p-8 ring-1 ring-gray-200 xl:p-10
                ${plan.featured ? 'bg-gray-900 ring-gray-900' : 'bg-white'}
                ${planIdx === 0 ? 'lg:z-10 lg:rounded-r-none' : ''}
                ${planIdx === plans.length - 1 ? 'lg:rounded-l-none' : ''}
                ${!plan.featured && planIdx > 0 ? '-mx-px lg:mx-0' : ''}
              `}
            >
              <div>
                <div className="flex items-center justify-between gap-x-4">
                  <h3
                    className={`
                      text-lg font-semibold leading-8
                      ${plan.featured ? 'text-white' : 'text-gray-900'}
                    `}
                  >
                    {plan.name}
                  </h3>
                  {plan.featured && (
                    <p className="rounded-full bg-indigo-500 px-2.5 py-1 text-xs font-semibold leading-5 text-white">
                      Most popular
                    </p>
                  )}
                </div>
                <p
                  className={`
                    mt-4 text-sm leading-6
                    ${plan.featured ? 'text-gray-300' : 'text-gray-600'}
                  `}
                >
                  {plan.description}
                </p>
                <p className="mt-6 flex items-baseline gap-x-1">
                  <span
                    className={`
                      text-4xl font-bold tracking-tight
                      ${plan.featured ? 'text-white' : 'text-gray-900'}
                    `}
                  >
                    {plan.price}
                  </span>
                  <span
                    className={`
                      text-sm font-semibold leading-6
                      ${plan.featured ? 'text-gray-300' : 'text-gray-600'}
                    `}
                  >
                    /month
                  </span>
                </p>
                <ul
                  role="list"
                  className="mt-8 space-y-3 text-sm leading-6"
                >
                  {plan.features.map((feature) => (
                    <li key={feature} className="flex gap-x-3">
                      <CheckIcon
                        className={`
                          h-6 w-5 flex-none
                          ${plan.featured ? 'text-white' : 'text-indigo-600'}
                        `}
                        aria-hidden="true"
                      />
                      <span
                        className={`
                          ${plan.featured ? 'text-gray-300' : 'text-gray-600'}
                        `}
                      >
                        {feature}
                      </span>
                    </li>
                  ))}
                </ul>
              </div>
              <a
                href="#"
                aria-describedby={plan.name}
                className={`
                  mt-8 block rounded-md px-3 py-2 text-center text-sm font-semibold leading-6 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2
                  ${
                    plan.featured
                      ? 'bg-indigo-500 text-white shadow-sm hover:bg-indigo-400 focus-visible:outline-indigo-500'
                      : 'bg-indigo-600 text-white shadow-sm hover:bg-indigo-500 focus-visible:outline-indigo-600'
                  }
                `}
              >
                Buy plan
              </a>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}
FAQ Accordion
tsx
// components/faq/accordion.tsx
'use client';

import { useState } from 'react';
import { PlusIcon, MinusIcon } from '@heroicons/react/20/solid';

const faqs = [
  {
    id: 1,
    question: "What's included in the free plan?",
    answer:
      "The free plan includes basic components, up to 3 projects, and community support. It's perfect for getting started.",
  },
  {
    id: 2,
    question: "How do I upgrade my plan?",
    answer:
      "You can upgrade from your account dashboard. All upgrades are prorated and take effect immediately.",
  },
  {
    id: 3,
    question: "Can I cancel my subscription anytime?",
    answer:
      "Yes, you can cancel anytime. You'll continue to have access until the end of your billing period.",
  },
  {
    id: 4,
    question: "Do you offer team discounts?",
    answer:
      "Yes! We offer significant discounts for teams of 5 or more. Contact our sales team for custom pricing.",
  },
  {
    id: 5,
    question: "Is there a free trial?",
    answer:
      "All paid plans come with a 14-day free trial. No credit card required to start your trial.",
  },
  {
    id: 6,
    question: "What payment methods do you accept?",
    answer:
      "We accept all major credit cards, PayPal, and wire transfers for enterprise customers.",
  },
];

export function FAQAccordion() {
  const [openItems, setOpenItems] = useState<number[]>([]);

  const toggleItem = (id: number) => {
    setOpenItems((prev) =>
      prev.includes(id) ? prev.filter((item) => item !== id) : [...prev, id]
    );
  };

  return (
    <div className="bg-white">
      <div className="mx-auto max-w-7xl px-6 py-24 sm:py-32 lg:px-8 lg:py-40">
        <div className="mx-auto max-w-4xl divide-y divide-gray-900/10">
          <h2 className="text-2xl font-bold leading-10 tracking-tight text-gray-900">
            Frequently asked questions
          </h2>
          <dl className="mt-10 space-y-6 divide-y divide-gray-900/10">
            {faqs.map((faq) => (
              <div key={faq.id} className="pt-6">
                <dt>
                  <button
                    onClick={() => toggleItem(faq.id)}
                    className="flex w-full items-start justify-between text-left text-gray-900"
                  >
                    <span className="text-base font-semibold leading-7">
                      {faq.question}
                    </span>
                    <span className="ml-6 flex h-7 items-center">
                      {openItems.includes(faq.id) ? (
                        <MinusIcon className="h-6 w-6" aria-hidden="true" />
                      ) : (
                        <PlusIcon className="h-6 w-6" aria-hidden="true" />
                      )}
                    </span>
                  </button>
                </dt>
                {openItems.includes(faq.id) && (
                  <dd className="mt-2 pr-12">
                    <p className="text-base leading-7 text-gray-600">
                      {faq.answer}
                    </p>
                  </dd>
                )}
              </div>
            ))}
          </dl>
        </div>
      </div>
    </div>
  );
}
CTA Section
tsx
// components/cta/section.tsx
export function CTASection() {
  return (
    <div className="relative isolate overflow-hidden bg-gray-900 px-6 py-24 sm:py-32 lg:px-8">
      <img
        src="https://images.unsplash.com/photo-1521737604893-d14cc237f11d?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=2830&q=80&blend=111827&sat=-100&exp=15&blend-mode=multiply"
        alt=""
        className="absolute inset-0 -z-10 h-full w-full object-cover"
      />
      <div className="hidden sm:absolute sm:-top-10 sm:right-1/2 sm:-z-10 sm:mr-10 sm:block sm:transform-gpu sm:blur-3xl">
        <div
          className="aspect-[1097/845] w-[68.5625rem] bg-gradient-to-tr from-[#ff4694] to-[#776fff] opacity-20"
          style={{
            clipPath:
              'polygon(74.1% 44.1%, 100% 61.6%, 97.5% 26.9%, 85.5% 0.1%, 80.7% 2%, 72.5% 32.5%, 60.2% 62.4%, 52.4% 68.1%, 47.5% 58.3%, 45.2% 34.5%, 27.5% 76.7%, 0.1% 64.9%, 17.9% 100%, 27.6% 76.8%, 76.1% 97.7%, 74.1% 44.1%)',
          }}
        />
      </div>
      <div className="absolute -top-52 left-1/2 -z-10 -translate-x-1/2 transform-gpu blur-3xl sm:top-[-28rem] sm:ml-16 sm:translate-x-0 sm:transform-gpu">
        <div
          className="aspect-[1097/845] w-[68.5625rem] bg-gradient-to-tr from-[#ff4694] to-[#776fff] opacity-20"
          style={{
            clipPath:
              'polygon(74.1% 44.1%, 100% 61.6%, 97.5% 26.9%, 85.5% 0.1%, 80.7% 2%, 72.5% 32.5%, 60.2% 62.4%, 52.4% 68.1%, 47.5% 58.3%, 45.2% 34.5%, 27.5% 76.7%, 0.1% 64.9%, 17.9% 100%, 27.6% 76.8%, 76.1% 97.7%, 74.1% 44.1%)',
          }}
        />
      </div>
      <div className="mx-auto max-w-2xl text-center">
        <h2 className="text-3xl font-bold tracking-tight text-white sm:text-4xl">
          Boost your productivity.
          <br />
          Start using our app today.
        </h2>
        <p className="mx-auto mt-6 max-w-xl text-lg leading-8 text-gray-300">
          Incididunt sint fugiat pariatur cupidatat consectetur sit cillum anim
          id veniam aliqua proident excepteur commodo do ea.
        </p>
        <div className="mt-10 flex items-center justify-center gap-x-6">
          <a
            href="#"
            className="rounded-md bg-white px-3.5 py-2.5 text-sm font-semibold text-gray-900 shadow-sm hover:bg-gray-100 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-white"
          >
            Get started
          </a>
          <a
            href="#"
            className="text-sm font-semibold leading-6 text-white"
          >
            Learn more <span aria-hidden="true">→</span>
          </a>
        </div>
      </div>
    </div>
  );
}
Footer Variants
tsx
// components/footer/variants.tsx
export function FooterDefault() {
  return (
    <footer className="bg-white">
      <div className="mx-auto max-w-7xl overflow-hidden px-6 py-20 sm:py-24 lg:px-8">
        <nav
          className="-mb-6 columns-2 sm:flex sm:justify-center sm:space-x-12"
          aria-label="Footer"
        >
          <div className="pb-6">
            <a
              href="#"
              className="text-sm leading-6 text-gray-600 hover:text-gray-900"
            >
              About
            </a>
          </div>
          <div className="pb-6">
            <a
              href="#"
              className="text-sm leading-6 text-gray-600 hover:text-gray-900"
            >
              Blog
            </a>
          </div>
          <div className="pb-6">
            <a
              href="#"
              className="text-sm leading-6 text-gray-600 hover:text-gray-900"
            >
              Jobs
            </a>
          </div>
          <div className="pb-6">
            <a
              href="#"
              className="text-sm leading-6 text-gray-600 hover:text-gray-900"
            >
              Press
            </a>
          </div>
          <div className="pb-6">
            <a
              href="#"
              className="text-sm leading-6 text-gray-600 hover:text-gray-900"
            >
              Accessibility
            </a>
          </div>
          <div className="pb-6">
            <a
              href="#"
              className="text-sm leading-6 text-gray-600 hover:text-gray-900"
            >
              Partners
            </a>
          </div>
        </nav>
        <div className="mt-10 flex justify-center space-x-10">
          <a href="#" className="text-gray-400 hover:text-gray-500">
            <span className="sr-only">Facebook</span>
            <FacebookIcon className="h-6 w-6" aria-hidden="true" />
          </a>
          <a href="#" className="text-gray-400 hover:text-gray-500">
            <span className="sr-only">Instagram</span>
            <InstagramIcon className="h-6 w-6" aria-hidden="true" />
          </a>
          <a href="#" className="text-gray-400 hover:text-gray-500">
            <span className="sr-only">Twitter</span>
            <TwitterIcon className="h-6 w-6" aria-hidden="true" />
          </a>
          <a href="#" className="text-gray-400 hover:text-gray-500">
            <span className="sr-only">GitHub</span>
            <GithubIcon className="h-6 w-6" aria-hidden="true" />
          </a>
          <a href="#" className="text-gray-400 hover:text-gray-500">
            <span className="sr-only">YouTube</span>
            <YoutubeIcon className="h-6 w-6" aria-hidden="true" />
          </a>
        </div>
        <p className="mt-10 text-center text-xs leading-5 text-gray-500">
          &copy; 2024 Your Company, Inc. All rights reserved.
        </p>
      </div>
    </footer>
  );
}
§ MARKETING PAGES
About Page Template
tsx
// app/about/page.tsx
import Image from 'next/image';

export default function AboutPage() {
  return (
    <div className="bg-white">
      <main>
        {/* Hero */}
        <div className="relative isolate">
          <div className="absolute inset-x-0 -top-40 -z-10 transform-gpu overflow-hidden blur-3xl sm:-top-80">
            <div
              className="relative left-[calc(50%-11rem)] aspect-[1155/678] w-[36.125rem] -translate-x-1/2 rotate-[30deg] bg-gradient-to-tr from-[#ff80b5] to-[#9089fc] opacity-30 sm:left-[calc(50%-30rem)] sm:w-[72.1875rem]"
              style={{
                clipPath:
                  'polygon(74.1% 44.1%, 100% 61.6%, 97.5% 26.9%, 85.5% 0.1%, 80.7% 2%, 72.5% 32.5%, 60.2% 62.4%, 52.4% 68.1%, 47.5% 58.3%, 45.2% 34.5%, 27.5% 76.7%, 0.1% 64.9%, 17.9% 100%, 27.6% 76.8%, 76.1% 97.7%, 74.1% 44.1%)',
              }}
            />
          </div>
          <div className="mx-auto max-w-7xl px-6 py-24 sm:py-32 lg:px-8">
            <div className="mx-auto max-w-2xl lg:mx-0">
              <h1 className="text-4xl font-bold tracking-tight text-gray-900 sm:text-6xl">
                We're changing the way people build websites
              </h1>
              <p className="mt-6 text-lg leading-8 text-gray-600">
                Anim aute id magna aliqua ad ad non deserunt sunt. Qui irure qui
                lorem cupidatat commodo. Elit sunt amet fugiat veniam occaecat
                fugiat aliqua.
              </p>
            </div>
          </div>
        </div>

        {/* Timeline */}
        <div className="mx-auto -mt-12 max-w-7xl px-6 lg:px-8">
          <div className="mx-auto grid max-w-2xl grid-cols-1 gap-8 overflow-hidden lg:mx-0 lg:max-w-none lg:grid-cols-4">
            {[
              {
                year: '2018',
                title: 'Company founded',
                description:
                  'Started with a vision to simplify web development for everyone.',
              },
              {
                year: '2020',
                title: 'First major release',
                description:
                  'Launched our initial component library with 50+ components.',
              },
              {
                year: '2022',
                title: '10,000 customers',
                description:
                  'Reached a major milestone with thousands of happy users.',
              },
              {
                year: '2024',
                title: 'Global expansion',
                description:
                  'Opened offices in 3 new countries and expanded our team.',
              },
            ].map((item) => (
              <div key={item.year}>
                <time
                  dateTime={item.year}
                  className="flex items-center text-sm font-semibold leading-6 text-indigo-600"
                >
                  <svg
                    viewBox="0 0 4 4"
                    className="mr-4 h-1 w-1 flex-none"
                    aria-hidden="true"
                  >
                    <circle cx={2} cy={2} r={2} fill="currentColor" />
                  </svg>
                  {item.year}
                  <div
                    className="absolute -ml-2 h-px w-screen -translate-x-full bg-gray-900/10 sm:-ml-4 lg:static lg:-mr-6 lg:ml-8 lg:w-auto lg:flex-auto lg:translate-x-0"
                    aria-hidden="true"
                  />
                </time>
                <p className="mt-6 text-lg font-semibold leading-8 tracking-tight text-gray-900">
                  {item.title}
                </p>
                <p className="mt-1 text-base leading-7 text-gray-600">
                  {item.description}
                </p>
              </div>
            ))}
          </div>
        </div>

        {/* Team */}
        <div className="mx-auto mt-32 max-w-7xl px-6 sm:mt-40 lg:px-8">
          <div className="mx-auto max-w-2xl lg:mx-0">
            <h2 className="text-3xl font-bold tracking-tight text-gray-900 sm:text-4xl">
              Our team
            </h2>
            <p className="mt-6 text-lg leading-8 text-gray-600">
              We're a dynamic group of individuals who are passionate about
              what we do and dedicated to delivering the best results for our
              clients.
            </p>
          </div>
          <ul
            role="list"
            className="mx-auto mt-20 grid max-w-2xl grid-cols-2 gap-x-8 gap-y-16 text-center sm:grid-cols-3 md:grid-cols-4 lg:mx-0 lg:max-w-none lg:grid-cols-5 xl:grid-cols-6"
          >
            {team.map((person) => (
              <li key={person.name}>
                <Image
                  className="mx-auto h-24 w-24 rounded-full"
                  src={person.imageUrl}
                  alt={person.name}
                  width={96}
                  height={96}
                />
                <h3 className="mt-6 text-base font-semibold leading-7 tracking-tight text-gray-900">
                  {person.name}
                </h3>
                <p className="text-sm leading-6 text-gray-600">
                  {person.role}
                </p>
              </li>
            ))}
          </ul>
        </div>

        {/* Values */}
        <div className="mx-auto mt-32 max-w-7xl px-6 lg:px-8">
          <div className="mx-auto max-w-2xl lg:text-center">
           

════════════════════════════════════════════════════════════
FIGMA CATALOG: WEBBY-37-WEBSITE-TEMPLATE
Prompt ID: 37 / 48
Parte: 2
Exported: 2026-02-06T13:39:25.306Z
Characters: 4724
════════════════════════════════════════════════════════════

href}>
                        <span className="absolute inset-0" />
                        {release.title}
                      </a>
                    </h3>
                    <p className="mt-5 text-sm leading-6 text-gray-600">
                      {release.description}
                    </p>
                  </div>
                  <div className="mt-6 flex border-t border-gray-900/5 pt-6">
                    <div className="relative flex items-center gap-x-4">
                      <Image
                        src={release.author.imageUrl}
                        alt={release.author.name}
                        width={40}
                        height={40}
                        className="h-10 w-10 rounded-full bg-gray-50"
                      />
                      <div className="text-sm leading-6">
                        <p className="font-semibold text-gray-900">
                          <a href={release.author.href}>
                            <span className="absolute inset-0" />
                            {release.author.name}
                          </a>
                        </p>
                        <p className="text-gray-600">{release.author.role}</p>
                      </div>
                    </div>
                  </div>
                </div>
              </article>
            ))}
          </div>
        </div>
      </div>

      {/* Media Coverage */}
      <div id="media-coverage" className="mx-auto mt-32 max-w-7xl px-6 lg:px-8">
        <div className="mx-auto max-w-2xl lg:max-w-none">
          <h2 className="text-3xl font-bold tracking-tight text-gray-900">
            Media coverage
          </h2>
          <div className="mt-16 grid grid-cols-1 gap-8 sm:grid-cols-2 lg:grid-cols-3">
            {mediaCoverage.map((coverage) => (
              <div
                key={coverage.id}
                className="rounded-2xl bg-gray-50 p-8 hover:bg-gray-100"
              >
                <div className="flex items-center gap-x-4">
                  <div className="flex h-12 w-12 items-center justify-center rounded-lg bg-white">
                    <Image
                      src={coverage.logo}
                      alt={coverage.outlet}
                      width={32}
                      height={32}
                      className="h-8 w-8"
                    />
                  </div>
                  <div>
                    <h3 className="text-lg font-semibold text-gray-900">
                      {coverage.outlet}
                    </h3>
                    <p className="text-sm text-gray-600">{coverage.date}</p>
                  </div>
                </div>
                <p className="mt-4 text-gray-600">{coverage.title}</p>
                <a
                  href={coverage.link}
                  className="mt-4 inline-flex items-center text-sm font-semibold text-indigo-600"
                >
                  Read article <span aria-hidden="true">→</span>
                </a>
              </div>
            ))}
          </div>
        </div>
      </div>

      {/* Press Kit */}
      <div id="press-kit" className="mx-auto mt-32 max-w-7xl px-6 lg:px-8">
        <div className="mx-auto max-w-2xl lg:max-w-none">
          <div className="rounded-2xl bg-gray-900 px-8 py-12 sm:px-12 sm:py-16">
            <div className="mx-auto max-w-2xl">
              <h2 className="text-3xl font-bold tracking-tight text-white sm:text-4xl">
                Press kit
              </h2>
              <p className="mt-4 text-lg leading-8 text-gray-300">
                Download logos, brand assets, and company information.
              </p>
              <div className="mt-8 grid grid-cols-1 gap-6 sm:grid-cols-2">
                {pressKit.map((item) => (
                  <a
                    key={item.name}
                    href={item.href}
                    className="rounded-lg bg-white/10 p-6 hover:bg-white/20"
                  >
                    <div className="flex items-center gap-x-4">
                      <item.icon className="h-8 w-8 text-white" aria-hidden="true" />
                      <div>
                        <h3 className="text-lg font-semibold text-white">
                          {item.name}
                        </h3>
                        <p className="mt-1 text-sm text-gray-300">
                          {item.description}
                        </p>
                      </div>
                    </div>
                  </a>
                ))}
              </div>
            </div>
          </div>
        </div>
      </div>

      {/* Contact */}
      <div id="contact" className="mx-auto mt-32 max-w-7xl px-6 lg:px