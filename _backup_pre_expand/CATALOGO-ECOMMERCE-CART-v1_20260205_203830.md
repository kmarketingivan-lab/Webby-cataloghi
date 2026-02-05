# CATALOGO-ECOMMERCE-CART-v1

> **Dominio**: E-Commerce
> **Stack**: Next.js, React, TypeScript, shadcn/ui, Prisma, tRPC, Zod
> **Versione**: 1.0
> **Data**: 2026-02-04

---

## 1. INDICE

| # | Sezione | Path |
|---|---------|------|
| 1 | [prisma/schema-ecommerce-cart.prisma](#1-prisma-schema-ecommerce-cart-prisma) | `prisma/schema-ecommerce-cart.prisma` |
| 2 | [src/server/services/cart-service.ts](#2-src-server-services-cart-service-ts) | `src/server/services/cart-service.ts` |
| 3 | [src/server/services/checkout-service.ts](#3-src-server-services-checkout-service-ts) | `src/server/services/checkout-service.ts` |
| 4 | [src/server/trpc/routers/cart.ts](#4-src-server-trpc-routers-cart-ts) | `src/server/trpc/routers/cart.ts` |
| 5 | [src/server/trpc/routers/checkout.ts](#5-src-server-trpc-routers-checkout-ts) | `src/server/trpc/routers/checkout.ts` |
| 6 | [src/lib/validations/cart.ts](#6-src-lib-validations-cart-ts) | `src/lib/validations/cart.ts` |
| 7 | [src/hooks/use-cart.ts](#7-src-hooks-use-cart-ts) | `src/hooks/use-cart.ts` |
| 8 | [src/hooks/use-checkout.ts](#8-src-hooks-use-checkout-ts) | `src/hooks/use-checkout.ts` |
| 9 | [src/components/cart/cart-drawer.tsx](#9-src-components-cart-cart-drawer-tsx) | `src/components/cart/cart-drawer.tsx` |
| 10 | [src/components/cart/cart-item.tsx](#10-src-components-cart-cart-item-tsx) | `src/components/cart/cart-item.tsx` |
| 11 | [src/components/cart/cart-summary.tsx](#11-src-components-cart-cart-summary-tsx) | `src/components/cart/cart-summary.tsx` |
| 12 | [src/components/cart/cart-icon.tsx](#12-src-components-cart-cart-icon-tsx) | `src/components/cart/cart-icon.tsx` |
| 13 | [src/components/checkout/checkout-form.tsx](#13-src-components-checkout-checkout-form-tsx) | `src/components/checkout/checkout-form.tsx` |
| 14 | [src/components/checkout/address-form.tsx](#14-src-components-checkout-address-form-tsx) | `src/components/checkout/address-form.tsx` |
| 15 | [src/components/checkout/shipping-options.tsx](#15-src-components-checkout-shipping-options-tsx) | `src/components/checkout/shipping-options.tsx` |

---

## 1. prisma/schema-ecommerce-cart.prisma

```prisma
// Questo schema assume l'esistenza di User, Product, ProductVariant e Coupon
// da un modulo precedente o da un'altra parte del sistema.
// Vengono inclusi qui per completezza delle relazioni.

model User {
  id            String          @id @default(cuid())
  email         String          @unique
  name          String?
  carts         Cart[]
  orders        Order[]
  addresses     Address[]
  createdAt     DateTime        @default(now())
  updatedAt     DateTime        @updatedAt
}

model Product {
  id            String          @id @default(cuid())
  name          String
  slug          String          @unique
  description   String?
  price         Decimal         @db.Decimal(10, 2)
  stock         Int             @default(0)
  variants      ProductVariant[]
  cartItems     CartItem[]
  orderItems    OrderItem[]
  createdAt     DateTime        @default(now())
  updatedAt     DateTime        @updatedAt
}

model ProductVariant {
  id            String          @id @default(cuid())
  productId     String
  product       Product         @relation(fields: [productId], references: [id], onDelete: Cascade)
  name          String          // e.g., "Size: M, Color: Red"
  sku           String          @unique
  priceModifier Decimal?        @db.Decimal(10, 2) // e.g., +5.00 for XL size
  stock         Int             @default(0)
  cartItems     CartItem[]
  orderItems    OrderItem[]
  createdAt     DateTime        @default(now())
  updatedAt     DateTime        @updatedAt
}

model Coupon {
  id            String          @id @default(cuid())
  code          String          @unique
  description   String?
  discountType  DiscountType
  discountValue Decimal         @db.Decimal(10, 2)
  minAmount     Decimal?        @db.Decimal(10, 2)
  maxUses       Int?
  uses          Int             @default(0)
  expiresAt     DateTime?
  isActive      Boolean         @default(true)
  carts         Cart[]
  orders        Order[]
  createdAt     DateTime        @default(now())
  updatedAt     DateTime        @updatedAt
}

enum DiscountType {
  PERCENTAGE
  FIXED_AMOUNT
}

model Cart {
  id          String      @id @default(cuid())
  userId      String?     @unique // Un utente può avere un solo carrello attivo
  user        User?       @relation(fields: [userId], references: [id])
  sessionId   String?     @unique // Per guest cart, un guest può avere un solo carrello attivo
  
  items       CartItem[]
  
  // Totals (cached, updated on item change)
  subtotal    Decimal     @default(0) @db.Decimal(10, 2)
  discount    Decimal     @default(0) @db.Decimal(10, 2)
  tax         Decimal     @default(0) @db.Decimal(10, 2)
  shipping    Decimal     @default(0) @db.Decimal(10, 2)
  total       Decimal     @default(0) @db.Decimal(10, 2)
  
  // Discount code applied
  couponId    String?
  coupon      Coupon?     @relation(fields: [couponId], references: [id])
  
  expiresAt   DateTime?   // Per carrelli guest o carrelli inattivi
  createdAt   DateTime    @default(now())
  updatedAt   DateTime    @updatedAt
  
  @@index([userId])
  @@index([sessionId])
  @@index([expiresAt])
}

model CartItem {
  id          String          @id @default(cuid())
  cartId      String
  cart        Cart            @relation(fields: [cartId], references: [id], onDelete: Cascade)
  
  productId   String
  product     Product         @relation(fields: [productId], references: [id])
  variantId   String?
  variant     ProductVariant? @relation(fields: [variantId], references: [id])
  
  quantity    Int
  price       Decimal         @db.Decimal(10, 2)  // Price at time of adding (including variant modifier)
  
  // Stock reservation (optional, for more advanced stock management)
  reservationId String?         // ID della prenotazione di stock esterna
  reservedUntil DateTime?       // Data fino a cui lo stock è riservato
  
  createdAt   DateTime        @default(now())
  updatedAt   DateTime        @updatedAt
  
  @@unique([cartId, productId, variantId]) // Unico per carrello, prodotto e variante
  @@index([cartId])
  @@index([productId])
  @@index([variantId])
}

// Modelli per il Checkout e l'Ordine (minimali per le relazioni)
model CheckoutSession {
  id                  String              @id @default(cuid())
  cartId              String              @unique // Ogni sessione di checkout è legata a un carrello
  cart                Cart                @relation(fields: [cartId], references: [id], onDelete: Cascade)
  userId              String?
  user                User?               @relation(fields: [userId], references: [id])
  
  currentStep         CheckoutStep        @default(SHIPPING_ADDRESS)
  
  shippingAddressId   String?
  shippingAddress     Address?            @relation("ShippingAddress", fields: [shippingAddressId], references: [id])
  
  billingAddressId    String?
  billingAddress      Address?            @relation("BillingAddress", fields: [billingAddressId], references: [id])
  
  shippingMethodId    String?             // ID del metodo di spedizione selezionato
  shippingMethodName  String?
  shippingMethodPrice Decimal?            @db.Decimal(10, 2)
  
  paymentMethodId     String?             // ID del metodo di pagamento (e.g., Stripe payment method ID)
  paymentIntentId     String?             // ID del Payment Intent di Stripe o transazione gateway
  
  // Totals (snapshot from cart, potentially adjusted by shipping/tax)
  subtotal            Decimal             @db.Decimal(10, 2)
  discount            Decimal             @db.Decimal(10, 2)
  tax                 Decimal             @db.Decimal(10, 2)
  shipping            Decimal             @db.Decimal(10, 2)
  total               Decimal             @db.Decimal(10, 2)
  
  expiresAt           DateTime            // Scadenza della sessione di checkout
  createdAt           DateTime            @default(now())
  updatedAt           DateTime            @updatedAt
  
  @@index([userId])
  @@index([expiresAt])
}

enum CheckoutStep {
  SHIPPING_ADDRESS
  BILLING_ADDRESS
  SHIPPING_METHOD
  PAYMENT
  REVIEW
  COMPLETED
}

model Address {
  id                  String              @id @default(cuid())
  userId              String?
  user                User?               @relation(fields: [userId], references: [id])
  
  firstName           String
  lastName            String
  company             String?
  address1            String
  address2            String?
  city                String
  state               String?
  zip                 String
  country             String
  phone               String?
  
  isDefaultShipping   Boolean             @default(false)
  isDefaultBilling    Boolean             @default(false)
  
  shippingCheckouts   CheckoutSession[]   @relation("ShippingAddress")
  billingCheckouts    CheckoutSession[]   @relation("BillingAddress")
  
  shippingOrders      Order[]             @relation("ShippingAddress")
  billingOrders       Order[]             @relation("BillingAddress")
  
  createdAt           DateTime            @default(now())
  updatedAt           DateTime            @updatedAt
}

model Order {
  id                  String              @id @default(cuid())
  userId              String?
  user                User?               @relation(fields: [userId], references: [id])
  
  status              OrderStatus         @default(PENDING)
  
  orderNumber         String              @unique // Numero d'ordine leggibile
  
  shippingAddressId   String
  shippingAddress     Address             @relation("ShippingAddress", fields: [shippingAddressId], references: [id])
  
  billingAddressId    String
  billingAddress      Address             @relation("BillingAddress", fields: [billingAddressId], references: [id])
  
  items               OrderItem[]
  
  // Totals (snapshot at time of order)
  subtotal            Decimal             @db.Decimal(10, 2)
  discount            Decimal             @db.Decimal(10, 2)
  tax                 Decimal             @db.Decimal(10, 2)
  shipping            Decimal             @db.Decimal(10, 2)
  total               Decimal             @db.Decimal(10, 2)
  
  couponId            String?
  coupon              Coupon?             @relation(fields: [couponId], references: [id])
  
  paymentMethod       String?             // e.g., "Stripe", "PayPal"
  paymentStatus       PaymentStatus       @default(PENDING)
  paymentIntentId     String?             // ID della transazione del gateway di pagamento
  
  shippingMethod      String?
  trackingNumber      String?
  
  createdAt           DateTime            @default(now())
  updatedAt           DateTime            @updatedAt
  
  @@index([userId])
  @@index([status])
}

enum OrderStatus {
  PENDING
  PROCESSING
  SHIPPED
  DELIVERED
  CANCELLED
  REFUNDED
}

enum PaymentStatus {
  PENDING
  PAID
  FAILED
  REFUNDED
  PARTIALLY_REFUNDED
}

model OrderItem {
  id          String          @id @default(cuid())
  orderId     String
  order       Order           @relation(fields: [orderId], references: [id], onDelete: Cascade)
  
  productId   String
  product     Product         @relation(fields: [productId], references: [id])
  variantId   String?
  variant     ProductVariant? @relation(fields: [variantId], references: [id])
  
  quantity    Int
  price       Decimal         @db.Decimal(10, 2)  // Price at time of order
  
  createdAt   DateTime        @default(now())
  updatedAt   DateTime        @updatedAt
  
  @@unique([orderId, productId, variantId])
  @@index([orderId])
  @@index([productId])
  @@index([variantId])
}
```

---

## 2. src/server/services/cart-service.ts

```typescript
import { PrismaClient, Cart, CartItem, Product, ProductVariant, Coupon, DiscountType, Address } from '@prisma/client';
import { Decimal } from 'decimal.js'; // Assumiamo l'uso di decimal.js per calcoli precisi

// Inizializza Prisma Client
const prisma = new PrismaClient();

// Tipi di input per il servizio
export interface AddItemInput {
  productId: string;
  variantId?: string;
  quantity: number;
}

export interface ShippingOption {
  id: string;
  name: string;
  price: Decimal;
  estimatedDelivery: string;
}

export interface ShippingRate {
  id: string;
  name: string;
  price: Decimal;
  currency: string;
  deliveryEstimate: string;
}

export class CartService {
  // --- Cart management ---

  /**
   * Recupera un carrello esistente per l'utente/sessione o ne crea uno nuovo.
   * @param userId ID dell'utente autenticato (opzionale).
   * @param sessionId ID della sessione per carrelli guest (opzionale).
   * @returns Il carrello trovato o creato.
   */
  async getOrCreateCart(userId?: string, sessionId?: string): Promise<Cart> {
    if (!userId && !sessionId) {
      throw new Error('Either userId or sessionId must be provided to get or create a cart.');
    }

    // Cerca un carrello esistente
    let cart: Cart | null = null;
    if (userId) {
      cart = await prisma.cart.findUnique({ where: { userId } });
    }
    if (!cart && sessionId) {
      cart = await prisma.cart.findUnique({ where: { sessionId } });
    }

    // Se non trovato, crea un nuovo carrello
    if (!cart) {
      const expiresAt = new Date();
      expiresAt.setDate(expiresAt.getDate() + 7); // Carrello guest scade in 7 giorni

      cart = await prisma.cart.create({
        data: {
          userId: userId || undefined,
          sessionId: sessionId || undefined,
          expiresAt: sessionId ? expiresAt : undefined, // Solo i carrelli guest hanno una scadenza
        },
      });
    }

    return cart;
  }

  /**
   * Recupera un carrello tramite il suo ID.
   * @param cartId ID del carrello.
   * @returns Il carrello o null se non trovato.
   */
  async getCart(cartId: string): Promise<Cart | null> {
    return prisma.cart.findUnique({
      where: { id: cartId },
      include: {
        items: {
          include: {
            product: true,
            variant: true,
          },
        },
        coupon: true,
      },
    });
  }

  /**
   * Recupera il carrello di un utente autenticato.
   * @param userId ID dell'utente.
   * @returns Il carrello dell'utente o null se non trovato.
   */
  async getUserCart(userId: string): Promise<Cart | null> {
    return prisma.cart.findUnique({
      where: { userId },
      include: {
        items: {
          include: {
            product: true,
            variant: true,
          },
        },
        coupon: true,
      },
    });
  }

  /**
   * Unisce un carrello guest a un carrello utente.
   * Se l'utente ha già un carrello, gli item del guest vengono aggiunti.
   * Il carrello guest viene eliminato.
   * @param guestSessionId ID della sessione del carrello guest.
   * @param userId ID dell'utente.
   * @returns Il carrello unito dell'utente.
   */
  async mergeGuestCart(guestSessionId: string, userId: string): Promise<Cart> {
    const guestCart = await prisma.cart.findUnique({
      where: { sessionId: guestSessionId },
      include: { items: true },
    });

    if (!guestCart) {
      // Se non c'è un carrello guest, restituisci semplicemente il carrello dell'utente o creane uno.
      return this.getOrCreateCart(userId);
    }

    let userCart = await prisma.cart.findUnique({
      where: { userId },
      include: { items: true },
    });

    if (!userCart) {
      // Se l'utente non ha un carrello, assegna il carrello guest all'utente.
      userCart = await prisma.cart.update({
        where: { id: guestCart.id },
        data: {
          userId: userId,
          sessionId: null, // Rimuovi la session ID del guest
          expiresAt: null, // Rimuovi la scadenza del guest
        },
      });
    } else {
      // Se l'utente ha già un carrello, unisci gli item.
      for (const guestItem of guestCart.items) {
        try {
          await this.addItem(userCart.id, {
            productId: guestItem.productId,
            variantId: guestItem.variantId || undefined,
            quantity: guestItem.quantity,
          });
        } catch (error) {
          console.error(`Failed to merge item ${guestItem.id} into user cart ${userCart.id}:`, error);
          // Potresti voler gestire questo errore in modo più robusto, e.g., notificare l'utente.
        }
      }
      // Elimina il carrello guest dopo l'unione.
      await this.deleteCart(guestCart.id);
    }

    // Ricalcola i totali del carrello utente dopo l'unione.
    return this.recalculateTotals(userCart.id);
  }

  /**
   * Rimuove tutti gli item da un carrello.
   * @param cartId ID del carrello.
   */
  async clearCart(cartId: string): Promise<void> {
    await prisma.cartItem.deleteMany({ where: { cartId } });
    await this.recalculateTotals(cartId); // Ricalcola i totali dopo aver svuotato
  }

  /**
   * Elimina un carrello e tutti i suoi item.
   * @param cartId ID del carrello.
   */
  async deleteCart(cartId: string): Promise<void> {
    await prisma.cart.delete({ where: { id: cartId } });
  }

  // --- Items ---

  /**
   * Aggiunge un item al carrello o aggiorna la quantità se già presente.
   * @param cartId ID del carrello.
   * @param data Dati dell'item da aggiungere.
   * @returns L'item del carrello aggiornato o creato.
   */
  async addItem(cartId: string, data: AddItemInput): Promise<CartItem> {
    const { productId, variantId, quantity } = data;

    if (quantity <= 0) {
      throw new Error('Quantity must be greater than zero.');
    }

    const product = await prisma.product.findUnique({
      where: { id: productId },
      include: { variants: true },
    });

    if (!product) {
      throw new Error('Product not found.');
    }

    let itemPrice = product.price;
    let stockAvailable = product.stock;

    if (variantId) {
      const variant = product.variants.find((v) => v.id === variantId);
      if (!variant) {
        throw new Error('Product variant not found.');
      }
      itemPrice = new Decimal(product.price).plus(variant.priceModifier || 0);
      stockAvailable = variant.stock;
    }

    const existingItem = await prisma.cartItem.findUnique({
      where: {
        cartId_productId_variantId: {
          cartId,
          productId,
          variantId: variantId || null,
        },
      },
    });

    let cartItem: CartItem;
    if (existingItem) {
      const newQuantity = existingItem.quantity + quantity;
      if (newQuantity > stockAvailable) {
        throw new Error(`Not enough stock for ${product.name}. Available: ${stockAvailable}`);
      }
      cartItem = await prisma.cartItem.update({
        where: { id: existingItem.id },
        data: { quantity: newQuantity },
      });
    } else {
      if (quantity > stockAvailable) {
        throw new Error(`Not enough stock for ${product.name}. Available: ${stockAvailable}`);
      }
      cartItem = await prisma.cartItem.create({
        data: {
          cartId,
          productId,
          variantId: variantId || null,
          quantity,
          price: itemPrice,
        },
      });
    }

    await this.recalculateTotals(cartId);
    return cartItem;
  }

  /**
   * Aggiorna la quantità di un item nel carrello.
   * @param cartId ID del carrello.
   * @param itemId ID dell'item del carrello.
   * @param quantity Nuova quantità.
   * @returns L'item del carrello aggiornato.
   */
  async updateItemQuantity(cartId: string, itemId: string, quantity: number): Promise<CartItem> {
    if (quantity <= 0) {
      return this.removeItem(cartId, itemId).then(() => {
        throw new Error('Item removed as quantity was set to zero or less.');
      });
    }

    const cartItem = await prisma.cartItem.findUnique({
      where: { id: itemId, cartId },
      include: { product: { include: { variants: true } }, variant: true },
    });

    if (!cartItem) {
      throw new Error('Cart item not found.');
    }

    const stockAvailable = cartItem.variant ? cartItem.variant.stock : cartItem.product.stock;
    if (quantity > stockAvailable) {
      throw new Error(`Not enough stock for ${cartItem.product.name}. Available: ${stockAvailable}`);
    }

    const updatedItem = await prisma.cartItem.update({
      where: { id: itemId },
      data: { quantity },
    });

    await this.recalculateTotals(cartId);
    return updatedItem;
  }

  /**
   * Rimuove un item dal carrello.
   * @param cartId ID del carrello.
   * @param itemId ID dell'item del carrello.
   */
  async removeItem(cartId: string, itemId: string): Promise<void> {
    await prisma.cartItem.delete({ where: { id: itemId, cartId } });
    await this.recalculateTotals(cartId);
  }

  // --- Calculations ---

  /**
   * Ricalcola tutti i totali del carrello (subtotal, discount, tax, shipping, total).
   * @param cartId ID del carrello.
   * @returns Il carrello aggiornato.
   */
  async recalculateTotals(cartId: string): Promise<Cart> {
    const cart = await prisma.cart.findUnique({
      where: { id: cartId },
      include: {
        items: {
          include: {
            product: true,
            variant: true,
          },
        },
        coupon: true,
      },
    });

    if (!cart) {
      throw new Error('Cart not found for recalculation.');
    }

    const subtotal = this.calculateSubtotal(cart.items);
    let discount = new Decimal(0);

    if (cart.couponId && cart.coupon) {
      const couponValidation = await this.validateCoupon(cart.coupon.code, cart);
      if (couponValidation.valid) {
        discount = this.calculateDiscount(subtotal, cart.coupon);
      } else {
        // Se il coupon non è più valido, rimuovilo dal carrello
        await prisma.cart.update({
          where: { id: cartId },
          data: { couponId: null },
        });
        cart.coupon = null; // Aggiorna l'oggetto cart in memoria
      }
    }

    // Per ora, tasse e spedizione sono 0. Verranno calcolate in checkout.
    // In un sistema reale, potresti voler calcolare una stima qui.
    const tax = new Decimal(0); // this.calculateTax(subtotal.minus(discount), shippingAddress);
    const shipping = new Decimal(0); // this.calculateShipping(cart.items, shippingAddress);

    const total = subtotal.minus(discount).plus(tax).plus(shipping);

    return prisma.cart.update({
      where: { id: cartId },
      data: {
        subtotal: subtotal,
        discount: discount,
        tax: tax,
        shipping: shipping,
        total: total,
      },
    });
  }

  /**
   * Calcola il subtotale degli item nel carrello.
   * @param items Lista di item del carrello.
   * @returns Il subtotale.
   */
  calculateSubtotal(items: (CartItem & { product: Product; variant: ProductVariant | null })[]): Decimal {
    return items.reduce((acc, item) => {
      const itemPrice = new Decimal(item.price);
      return acc.plus(itemPrice.times(item.quantity));
    }, new Decimal(0));
  }

  /**
   * Calcola lo sconto basato su un coupon.
   * @param subtotal Il subtotale attuale del carrello.
   * @param coupon Il coupon da applicare.
   * @returns L'importo dello sconto.
   */
  private calculateDiscount(subtotal: Decimal, coupon: Coupon): Decimal {
    let discountAmount = new Decimal(0);
    if (coupon.discountType === DiscountType.PERCENTAGE) {
      discountAmount = subtotal.times(coupon.discountValue).dividedBy(100);
    } else if (coupon.discountType === DiscountType.FIXED_AMOUNT) {
      discountAmount = new Decimal(coupon.discountValue);
    }
    // Assicurati che lo sconto non superi il subtotale
    return Decimal.min(discountAmount, subtotal);
  }

  /**
   * Calcola l'importo delle tasse. (Placeholder, logica complessa in un sistema reale)
   * @param taxableAmount Importo su cui calcolare le tasse.
   * @param shippingAddress Indirizzo di spedizione per calcoli basati sulla località.
   * @returns L'importo delle tasse.
   */
  async calculateTax(taxableAmount: Decimal, shippingAddress?: Address): Promise<Decimal> {
    // Logica complessa per il calcolo delle tasse basata su indirizzo, tipo di prodotto, ecc.
    // Per ora, restituisce 0.
    console.log('Calculating tax for:', taxableAmount, shippingAddress);
    return new Decimal(0);
  }

  /**
   * Calcola i costi di spedizione. (Placeholder, logica complessa in un sistema reale)
   * @param items Item nel carrello per calcoli basati su peso/dimensioni.
   * @param address Indirizzo di spedizione per calcoli basati sulla località.
   * @returns L'importo della spedizione.
   */
  async calculateShipping(items: CartItem[], address?: Address): Promise<Decimal> {
    // Logica complessa per il calcolo della spedizione basata su peso, dimensioni, destinazione, metodo.
    // Per ora, restituisce 0.
    console.log('Calculating shipping for:', items, address);
    return new Decimal(0);
  }

  // --- Coupons ---

  /**
   * Applica un coupon al carrello.
   * @param cartId ID del carrello.
   * @param code Codice del coupon.
   * @returns Il carrello aggiornato e l'importo dello sconto.
   */
  async applyCoupon(cartId: string, code: string): Promise<{ cart: Cart; discount: Decimal }> {
    const cart = await prisma.cart.findUnique({
      where: { id: cartId },
      include: { items: true },
    });
    if (!cart) {
      throw new Error('Cart not found.');
    }

    const coupon = await prisma.coupon.findUnique({ where: { code } });
    if (!coupon) {
      throw new Error('Coupon not found or invalid.');
    }

    const validation = await this.validateCoupon(code, cart);
    if (!validation.valid) {
      throw new Error(validation.error || 'Coupon is not valid.');
    }

    const updatedCart = await prisma.cart.update({
      where: { id: cartId },
      data: { couponId: coupon.id },
    });

    const recalculatedCart = await this.recalculateTotals(cartId);
    return { cart: recalculatedCart, discount: recalculatedCart.discount };
  }

  /**
   * Rimuove un coupon dal carrello.
   * @param cartId ID del carrello.
   * @returns Il carrello aggiornato.
   */
  async removeCoupon(cartId: string): Promise<Cart> {
    await prisma.cart.update({
      where: { id: cartId },
      data: { couponId: null },
    });
    return this.recalculateTotals(cartId);
  }

  /**
   * Valida un coupon per un dato carrello.
   * @param code Codice del coupon.
   * @param cart Il carrello su cui applicare il coupon.
   * @returns Oggetto con validità e eventuale errore.
   */
  async validateCoupon(code: string, cart: Cart & { items: CartItem[] }): Promise<{ valid: boolean; error?: string }> {
    const coupon = await prisma.coupon.findUnique({ where: { code } });

    if (!coupon || !coupon.isActive) {
      return { valid: false, error: 'Coupon is invalid or expired.' };
    }
    if (coupon.expiresAt && coupon.expiresAt < new Date()) {
      return { valid: false, error: 'Coupon has expired.' };
    }
    if (coupon.maxUses && coupon.uses >= coupon.maxUses) {
      return { valid: false, error: 'Coupon has reached its maximum uses.' };
    }

    const subtotal = this.calculateSubtotal(
      cart.items.map((item) => ({
        ...item,
        product: {} as Product, // Mock per il tipo, non usato nel calcolo del subtotal
        variant: {} as ProductVariant,
      }))
    );

    if (coupon.minAmount && subtotal.lessThan(coupon.minAmount)) {
      return { valid: false, error: `Minimum purchase of ${coupon.minAmount} required.` };
    }

    return { valid: true };
  }

  // --- Stock ---

  /**
   * Valida la disponibilità di stock per tutti gli item nel carrello.
   * @param cartId ID del carrello.
   * @returns Oggetto con validità e lista di item non disponibili.
   */
  async validateStock(cartId: string): Promise<{ valid: boolean; unavailableItems: CartItem[] }> {
    const cart = await prisma.cart.findUnique({
      where: { id: cartId },
      include: {
        items: {
          include: {
            product: true,
            variant: true,
          },
        },
      },
    });

    if (!cart) {
      throw new Error('Cart not found.');
    }

    const unavailableItems: CartItem[] = [];
    for (const item of cart.items) {
      const stockAvailable = item.variant ? item.variant.stock : item.product.stock;
      if (item.quantity > stockAvailable) {
        unavailableItems.push(item);
      }
    }

    return { valid: unavailableItems.length === 0, unavailableItems };
  }

  /**
   * Riserva lo stock per gli item nel carrello. (Logica placeholder)
   * In un sistema reale, questo interagirebbe con un sistema di gestione magazzino.
   * @param cartId ID del carrello.
   */
  async reserveStock(cartId: string): Promise<void> {
    const cart = await prisma.cart.findUnique({
      where: { id: cartId },
      include: { items: { include: { product: true, variant: true } } },
    });

    if (!cart) {
      throw new Error('Cart not found.');
    }

    const stockValidation = await this.validateStock(cartId);
    if (!stockValidation.valid) {
      throw new Error('Cannot reserve stock: some items are out of stock.');
    }

    // Simula la prenotazione dello stock e aggiorna gli item del carrello
    const reservedUntil = new Date();
    reservedUntil.setMinutes(reservedUntil.getMinutes() + 30); // Riserva per 30 minuti

    for (const item of cart.items) {
      // In un sistema reale, qui si chiamerebbe un servizio di stock esterno
      // e si salverebbe l'ID della prenotazione.
      await prisma.cartItem.update({
        where: { id: item.id },
        data: {
          reservationId: `RES-${item.id}-${Date.now()}`, // ID di prenotazione fittizio
          reservedUntil: reservedUntil,
        },
      });
      // Qui si decrementerebbe lo stock effettivo o si creerebbe una prenotazione nel sistema di magazzino.
      // Per semplicità, non modifichiamo lo stock del Product/ProductVariant qui,
      // ma lo faremmo al momento della creazione dell'ordine.
    }
    console.log(`Stock reserved for cart ${cartId} until ${reservedUntil.toISOString()}`);
  }

  /**
   * Rilascia lo stock precedentemente riservato. (Logica placeholder)
   * @param cartId ID del carrello.
   */
  async releaseStock(cartId: string): Promise<void> {
    const cart = await prisma.cart.findUnique({
      where: { id: cartId },
      include: { items: true },
    });

    if (!cart) {
      console.warn(`Attempted to release stock for non-existent cart ${cartId}.`);
      return;
    }

    for (const item of cart.items) {
      if (item.reservationId) {
        // In un sistema reale, qui si chiamerebbe un servizio di stock esterno
        // per rilasciare la prenotazione.
        await prisma.cartItem.update({
          where: { id: item.id },
          data: {
            reservationId: null,
            reservedUntil: null,
          },
        });
        console.log(`Released reservation ${item.reservationId} for item ${item.id}.`);
      }
    }
  }

  // --- Expiration ---

  /**
   * Pulisce i carrelli scaduti (tipicamente carrelli guest).
   * @returns Il numero di carrelli eliminati.
   */
  async cleanupExpiredCarts(): Promise<number> {
    const result = await prisma.cart.deleteMany({
      where: {
        expiresAt: {
          lt: new Date(),
        },
        userId: null, // Solo carrelli guest
      },
    });
    console.log(`Cleaned up ${result.count} expired guest carts.`);
    return result.count;
  }

  /**
   * Estende la data di scadenza di un carrello.
   * Utile per carrelli guest attivi.
   * @param cartId ID del carrello.
   */
  async extendCartExpiration(cartId: string): Promise<void> {
    const cart = await prisma.cart.findUnique({ where: { id: cartId } });
    if (!cart) {
      throw new Error('Cart not found.');
    }
    if (cart.userId) {
      // I carrelli utente non scadono, non è necessario estendere.
      return;
    }

    const newExpiresAt = new Date();
    newExpiresAt.setDate(newExpiresAt.getDate() + 7); // Estende di altri 7 giorni

    await prisma.cart.update({
      where: { id: cartId },
      data: { expiresAt: newExpiresAt },
    });
    console.log(`Extended expiration for cart ${cartId} to ${newExpiresAt.toISOString()}`);
  }
}
```

---

## 3. src/server/services/checkout-service.ts

```typescript
import {
  PrismaClient,
  Cart,
  CartItem,
  CheckoutSession,
  CheckoutStep,
  Address,
  Order,
  OrderItem,
  OrderStatus,
  PaymentStatus,
  Product,
  ProductVariant,
  Coupon,
} from '@prisma/client';
import { Decimal } from 'decimal.js'; // Assumiamo l'uso di decimal.js per calcoli precisi
import { CartService, ShippingRate } from './cart-service'; // Importa CartService e ShippingRate

// Inizializza Prisma Client
const prisma = new PrismaClient();
const cartService = new CartService(); // Istanzia il CartService

// Tipi di input e output
export interface AddressInput {
  firstName: string;
  lastName: string;
  company?: string;
  address1: string;
  address2?: string;
  city: string;
  state?: string;
  zip: string;
  country: string;
  phone?: string;
}

export interface UpdateCheckoutInput {
  currentStep?: CheckoutStep;
  shippingAddressId?: string;
  billingAddressId?: string;
  shippingMethodId?: string;
  shippingMethodName?: string;
  shippingMethodPrice?: Decimal;
  paymentMethodId?: string;
  paymentIntentId?: string;
  subtotal?: Decimal;
  discount?: Decimal;
  tax?: Decimal;
  shipping?: Decimal;
  total?: Decimal;
}

export interface ShippingMethod {
  id: string;
  name: string;
  description: string;
  price: Decimal;
  estimatedDelivery: string;
}

export interface PaymentResult {
  success: boolean;
  transactionId?: string;
  message?: string;
}

export class CheckoutService {
  // --- Checkout session ---

  /**
   * Crea una nuova sessione di checkout per un dato carrello.
   * @param cartId ID del carrello.
   * @returns La sessione di checkout creata.
   */
  async createCheckoutSession(cartId: string): Promise<CheckoutSession> {
    const cart = await prisma.cart.findUnique({
      where: { id: cartId },
      include: { items: true, coupon: true },
    });

    if (!cart) {
      throw new Error('Cart not found for checkout session creation.');
    }

    // Valida lo stock prima di creare la sessione di checkout
    const stockValidation = await cartService.validateStock(cartId);
    if (!stockValidation.valid) {
      const unavailableProductNames = stockValidation.unavailableItems
        .map((item) => item.productId)
        .join(', ');
      throw new Error(`Some items are out of stock: ${unavailableProductNames}`);
    }

    // Riserva lo stock (se implementato)
    await cartService.reserveStock(cartId);

    const expiresAt = new Date();
    expiresAt.setHours(expiresAt.getHours() + 1); // Sessione di checkout scade in 1 ora

    // Inizializza i totali dalla cart
    const initialTotals = await cartService.recalculateTotals(cartId);

    const existingSession = await prisma.checkoutSession.findUnique({ where: { cartId } });
    if (existingSession) {
      // Se esiste già una sessione per questo carrello, aggiornala
      return prisma.checkoutSession.update({
        where: { id: existingSession.id },
        data: {
          userId: cart.userId,
          currentStep: CheckoutStep.SHIPPING_ADDRESS,
          subtotal: initialTotals.subtotal,
          discount: initialTotals.discount,
          tax: initialTotals.tax,
          shipping: initialTotals.shipping,
          total: initialTotals.total,
          expiresAt,
        },
      });
    }

    return prisma.checkoutSession.create({
      data: {
        cartId,
        userId: cart.userId,
        currentStep: CheckoutStep.SHIPPING_ADDRESS,
        subtotal: initialTotals.subtotal,
        discount: initialTotals.discount,
        tax: initialTotals.tax,
        shipping: initialTotals.shipping,
        total: initialTotals.total,
        expiresAt,
      },
    });
  }

  /**
   * Recupera una sessione di checkout tramite il suo ID.
   * @param sessionId ID della sessione di checkout.
   * @returns La sessione di checkout o null se non trovata.
   */
  async getCheckoutSession(sessionId: string): Promise<CheckoutSession | null> {
    return prisma.checkoutSession.findUnique({
      where: { id: sessionId },
      include: {
        cart: {
          include: {
            items: {
              include: {
                product: true,
                variant: true,
              },
            },
            coupon: true,
          },
        },
        shippingAddress: true,
        billingAddress: true,
      },
    });
  }

  /**
   * Aggiorna i dati di una sessione di checkout.
   * @param sessionId ID della sessione di checkout.
   * @param data Dati da aggiornare.
   * @returns La sessione di checkout aggiornata.
   */
  async updateCheckoutSession(sessionId: string, data: UpdateCheckoutInput): Promise<CheckoutSession> {
    const session = await prisma.checkoutSession.findUnique({ where: { id: sessionId } });
    if (!session) {
      throw new Error('Checkout session not found.');
    }

    return prisma.checkoutSession.update({
      where: { id: sessionId },
      data: {
        ...data,
        expiresAt: new Date(new Date().setHours(new Date().getHours() + 1)), // Estende la scadenza ad ogni aggiornamento
      },
    });
  }

  // --- Steps ---

  /**
   * Imposta l'indirizzo di spedizione per la sessione di checkout.
   * @param sessionId ID della sessione di checkout.
   * @param address Dati dell'indirizzo.
   * @returns La sessione di checkout aggiornata.
   */
  async setShippingAddress(sessionId: string, address: AddressInput): Promise<CheckoutSession> {
    const session = await this.getCheckoutSession(sessionId);
    if (!session) {
      throw new Error('Checkout session not found.');
    }

    let existingAddress: Address | null = null;
    if (session.userId) {
      // Cerca un indirizzo esistente per l'utente con gli stessi dati
      existingAddress = await prisma.address.findFirst({
        where: {
          userId: session.userId,
          firstName: address.firstName,
          lastName: address.lastName,
          address1: address.address1,
          zip: address.zip,
          country: address.country,
        },
      });
    }

    let newAddress: Address;
    if (existingAddress) {
      newAddress = existingAddress;
    } else {
      newAddress = await prisma.address.create({
        data: {
          ...address,
          userId: session.userId || undefined,
        },
      });
    }

    // Ricalcola spedizione e tasse dopo aver impostato l'indirizzo
    const cartItems = session.cart?.items || [];
    const shippingRates = await this.calculateShippingRates(newAddress, cartItems);
    // Potresti voler selezionare un metodo di spedizione predefinito qui o lasciare che l'utente lo scelga.
    // Per ora, non aggiorniamo shippingMethodId/Price, lo faremo in setShippingMethod.

    const updatedSession = await prisma.checkoutSession.update({
      where: { id: sessionId },
      data: {
        shippingAddressId: newAddress.id,
        currentStep: CheckoutStep.BILLING_ADDRESS, // Passa al prossimo step
      },
      include: { shippingAddress: true, billingAddress: true, cart: true },
    });

    return updatedSession;
  }

  /**
   * Imposta l'indirizzo di fatturazione per la sessione di checkout.
   * @param sessionId ID della sessione di checkout.
   * @param address Dati dell'indirizzo.
   * @param sameAsShipping Se l'indirizzo di fatturazione è lo stesso di quello di spedizione.
   * @returns La sessione di checkout aggiornata.
   */
  async setBillingAddress(sessionId: string, address: AddressInput, sameAsShipping: boolean = false): Promise<CheckoutSession> {
    const session = await this.getCheckoutSession(sessionId);
    if (!session) {
      throw new Error('Checkout session not found.');
    }

    let billingAddressId: string | null = null;

    if (sameAsShipping && session.shippingAddressId) {
      billingAddressId = session.shippingAddressId;
    } else {
      let existingAddress: Address | null = null;
      if (session.userId) {
        existingAddress = await prisma.address.findFirst({
          where: {
            userId: session.userId,
            firstName: address.firstName,
            lastName: address.lastName,
            address1: address.address1,
            zip: address.zip,
            country: address.country,
          },
        });
      }

      let newAddress: Address;
      if (existingAddress) {
        newAddress = existingAddress;
      } else {
        newAddress = await prisma.address.create({
          data: {
            ...address,
            userId: session.userId || undefined,
          },
        });
      }
      billingAddressId = newAddress.id;
    }

    return prisma.checkoutSession.update({
      where: { id: sessionId },
      data: {
        billingAddressId: billingAddressId,
        currentStep: CheckoutStep.SHIPPING_METHOD, // Passa al prossimo step
      },
      include: { shippingAddress: true, billingAddress: true, cart: true },
    });
  }

  /**
   * Imposta il metodo di spedizione per la sessione di checkout.
   * @param sessionId ID della sessione di checkout.
   * @param methodId ID del metodo di spedizione selezionato.
   * @returns La sessione di checkout aggiornata.
   */
  async setShippingMethod(sessionId: string, methodId: string): Promise<CheckoutSession> {
    const session = await this.getCheckoutSession(sessionId);
    if (!session || !session.shippingAddress || !session.cart) {
      throw new Error('Checkout session, shipping address, or cart not found.');
    }

    const availableMethods = await this.calculateShippingRates(session.shippingAddress, session.cart.items);
    const selectedMethod = availableMethods.find((m) => m.id === methodId);

    if (!selectedMethod) {
      throw new Error('Invalid shipping method selected.');
    }

    // Aggiorna i totali con il costo di spedizione
    const currentSubtotal = new Decimal(session.subtotal);
    const currentDiscount = new Decimal(session.discount);
    const currentTax = new Decimal(session.tax); // Tasse potrebbero cambiare con la spedizione
    const shippingCost = new Decimal(selectedMethod.price);

    const newTotal = currentSubtotal.minus(currentDiscount).plus(currentTax).plus(shippingCost);

    return prisma.checkoutSession.update({
      where: { id: sessionId },
      data: {
        shippingMethodId: selectedMethod.id,
        shippingMethodName: selectedMethod.name,
        shippingMethodPrice: shippingCost,
        shipping: shippingCost,
        total: newTotal,
        currentStep: CheckoutStep.PAYMENT, // Passa al prossimo step
      },
      include: { shippingAddress: true, billingAddress: true, cart: true },
    });
  }

  /**
   * Imposta il metodo di pagamento per la sessione di checkout.
   * @param sessionId ID della sessione di checkout.
   * @param paymentMethodId ID del metodo di pagamento (e.g., Stripe Payment Method ID).
   * @returns La sessione di checkout aggiornata.
   */
  async setPaymentMethod(sessionId: string, paymentMethodId: string): Promise<CheckoutSession> {
    const session = await this.getCheckoutSession(sessionId);
    if (!session) {
      throw new Error('Checkout session not found.');
    }

    // Qui potresti voler validare il paymentMethodId con il gateway di pagamento
    // Per ora, lo salviamo direttamente.

    return prisma.checkoutSession.update({
      where: { id: sessionId },
      data: {
        paymentMethodId: paymentMethodId,
        currentStep: CheckoutStep.REVIEW, // Passa al prossimo step
      },
      include: { shippingAddress: true, billingAddress: true, cart: true },
    });
  }

  // --- Shipping options ---

  /**
   * Recupera le opzioni di spedizione disponibili per la sessione di checkout.
   * @param sessionId ID della sessione di checkout.
   * @returns Lista di opzioni di spedizione.
   */
  async getShippingOptions(sessionId: string): Promise<ShippingRate[]> {
    const session = await this.getCheckoutSession(sessionId);
    if (!session || !session.shippingAddress || !session.cart) {
      throw new Error('Checkout session, shipping address, or cart not found.');
    }

    return this.calculateShippingRates(session.shippingAddress, session.cart.items);
  }

  /**
   * Calcola le tariffe di spedizione disponibili per un dato indirizzo e item.
   * (Logica placeholder, in un sistema reale si integrerebbe con un API di spedizione)
   * @param address Indirizzo di spedizione.
   * @param items Item nel carrello.
   * @returns Lista di tariffe di spedizione.
   */
  async calculateShippingRates(address: Address, items: (CartItem & { product: Product; variant: ProductVariant | null })[]): Promise<ShippingRate[]> {
    // Logica complessa per calcolare le tariffe di spedizione
    // basata su peso, dimensioni, destinazione, corrieri, ecc.
    // Per ora, restituisce opzioni fittizie.
    console.log('Calculating shipping rates for:', address, items);

    const basePrice = new Decimal(10.00);
    const expressPrice = new Decimal(25.00);

    // Esempio: costo aggiuntivo per ogni 5 item
    const itemFactor = new Decimal(Math.floor(items.length / 5)).times(2.00);

    return [
      {
        id: 'standard',
        name: 'Standard Shipping',
        price: basePrice.plus(itemFactor),
        currency: 'EUR',
        deliveryEstimate: '5-7 business days',
      },
      {
        id: 'express',
        name: 'Express Shipping',
        price: expressPrice.plus(itemFactor),
        currency: 'EUR',
        deliveryEstimate: '1-2 business days',
      },
    ];
  }

  // --- Tax ---

  /**
   * Calcola le tasse per la sessione di checkout.
   * (Logica placeholder, in un sistema reale si integrerebbe con un API di tasse)
   * @param sessionId ID della sessione di checkout.
   * @returns L'importo delle tasse.
   */
  async calculateTax(sessionId: string): Promise<Decimal> {
    const session = await this.getCheckoutSession(sessionId);
    if (!session || !session.shippingAddress || !session.cart) {
      throw new Error('Checkout session, shipping address, or cart not found.');
    }

    const taxableAmount = new Decimal(session.subtotal).minus(session.discount).plus(session.shipping || 0);
    // Esempio: 22% di IVA per l'Italia
    if (session.shippingAddress.country === 'Italy') {
      return taxableAmount.times(0.22).toDecimalPlaces(2);
    }
    return new Decimal(0);
  }

  // --- Validation ---

  /**
   * Valida lo stato attuale della sessione di checkout per assicurarsi che tutti i dati necessari siano presenti.
   * @param sessionId ID della sessione di checkout.
   * @returns Oggetto con validità e lista di errori.
   */
  async validateCheckout(sessionId: string): Promise<{ valid: boolean; errors: string[] }> {
    const session = await this.getCheckoutSession(sessionId);
    if (!session) {
      return { valid: false, errors: ['Checkout session not found.'] };
    }

    const errors: string[] = [];

    if (!session.cart || session.cart.items.length === 0) {
      errors.push('Cart is empty.');
    }

    const stockValidation = await cartService.validateStock(session.cartId);
    if (!stockValidation.valid) {
      errors.push('Some items in your cart are out of stock.');
    }

    if (!session.shippingAddressId) {
      errors.push('Shipping address is required.');
    }
    if (!session.billingAddressId) {
      errors.push('Billing address is required.');
    }
    if (!session.shippingMethodId || !session.shippingMethodPrice) {
      errors.push('Shipping method is required.');
    }
    if (!session.paymentMethodId) {
      errors.push('Payment method is required.');
    }

    // Assicurati che i totali siano stati calcolati
    if (new Decimal(session.total).lessThanOrEqualTo(0)) {
      errors.push('Order total must be greater than zero.');
    }

    return { valid: errors.length === 0, errors };
  }

  // --- Process ---

  /**
   * Processa il checkout, creando l'ordine e gestendo il pagamento.
   * @param sessionId ID della sessione di checkout.
   * @returns L'ordine creato.
   */
  async processCheckout(sessionId: string): Promise<Order> {
    const validation = await this.validateCheckout(sessionId);
    if (!validation.valid) {
      throw new Error(`Checkout validation failed: ${validation.errors.join(', ')}`);
    }

    const session = await this.getCheckoutSession(sessionId);
    if (!session) {
      throw new Error('Checkout session not found during processing.');
    }

    // 1. Processa il pagamento
    const paymentResult = await this.processPayment(session);
    if (!paymentResult.success) {
      throw new Error(`Payment failed: ${paymentResult.message}`);
    }

    // 2. Crea l'ordine
    const order = await this.createOrderFromCheckout(session, paymentResult.transactionId);

    // 3. Decrementa lo stock dei prodotti
    await this.decrementProductStock(order.id);

    // 4. Rilascia lo stock riservato (se implementato)
    await cartService.releaseStock(session.cartId);

    // 5. Svuota il carrello
    await cartService.clearCart(session.cartId);

    // 6. Aggiorna lo stato della sessione di checkout
    await prisma.checkoutSession.update({
      where: { id: sessionId },
      data: { currentStep: CheckoutStep.COMPLETED },
    });

    // 7. Invia conferma ordine
    await this.sendOrderConfirmation(order);

    return order;
  }

  // --- Helpers ---

  /**
   * Crea un ordine dal checkout session.
   * @param session La sessione di checkout.
   * @param paymentIntentId ID della transazione di pagamento.
   * @returns L'ordine creato.
   */
  private async createOrderFromCheckout(session: CheckoutSession & { cart: (Cart & { items: (CartItem & { product: Product; variant: ProductVariant | null })[] }) | null; shippingAddress: Address | null; billingAddress: Address | null; }, paymentIntentId?: string): Promise<Order> {
    if (!session.cart || !session.shippingAddress || !session.billingAddress) {
      throw new Error('Missing data in checkout session to create order.');
    }

    const orderNumber = `ORD-${Date.now()}-${Math.random().toString(36).substr(2, 5).toUpperCase()}`;

    const order = await prisma.order.create({
      data: {
        userId: session.userId,
        orderNumber: orderNumber,
        shippingAddressId: session.shippingAddress.id,
        billingAddressId: session.billingAddress.id,
        subtotal: session.subtotal,
        discount: session.discount,
        tax: session.tax,
        shipping: session.shipping,
        total: session.total,
        couponId: session.cart.couponId,
        paymentMethod: session.paymentMethodId, // Potrebbe essere un nome più user-friendly
        paymentStatus: PaymentStatus.PAID,
        paymentIntentId: paymentIntentId,
        shippingMethod: session.shippingMethodName,
        status: OrderStatus.PROCESSING,
        items: {
          create: session.cart.items.map((item) => ({
            productId: item.productId,
            variantId: item.variantId,
            quantity: item.quantity,
            price: item.price,
          })),
        },
      },
      include: { items: true },
    });

    // Incrementa l'uso del coupon se presente
    if (session.cart.couponId) {
      await prisma.coupon.update({
        where: { id: session.cart.couponId },
        data: { uses: { increment: 1 } },
      });
    }

    return order;
  }

  /**
   * Simula il processo di pagamento con un gateway esterno.
   * @param session La sessione di checkout.
   * @returns Il risultato del pagamento.
   */
  private async processPayment(session: CheckoutSession): Promise<PaymentResult> {
    // Qui si integrerebbe con Stripe, PayPal, ecc.
    // Esempio fittizio:
    console.log(`Processing payment for total: ${session.total} with method: ${session.paymentMethodId}`);

    // Simula un ritardo o un errore casuale
    await new Promise((resolve) => setTimeout(resolve, 1000));

    const isSuccess = Math.random() > 0.1; // 90% di successo

    if (isSuccess) {
      return {
        success: true,
        transactionId: `TXN-${Date.now()}-${Math.random().toString(36).substr(2, 9).toUpperCase()}`,
        message: 'Payment successful.',
      };
    } else {
      return {
        success: false,
        message: 'Payment failed due to an unknown error. Please try again.',
      };
    }
  }

  /**
   * Decrementa lo stock dei prodotti dopo la creazione dell'ordine.
   * @param orderId ID dell'ordine.
   */
  private async decrementProductStock(orderId: string): Promise<void> {
    const order = await prisma.order.findUnique({
      where: { id: orderId },
      include: { items: { include: { product: true, variant: true } } },
    });

    if (!order) {
      throw new Error('Order not found for stock decrement.');
    }

    for (const item of order.items) {
      if (item.variantId && item.variant) {
        await prisma.productVariant.update({
          where: { id: item.variantId },
          data: { stock: { decrement: item.quantity } },
        });
      } else {
        await prisma.product.update({
          where: { id: item.productId },
          data: { stock: { decrement: item.quantity } },
        });
      }
    }
    console.log(`Stock decremented for order ${orderId}.`);
  }

  /**
   * Invia l'email di conferma dell'ordine. (Logica placeholder)
   * @param order L'ordine creato.
   */
  private async sendOrderConfirmation(order: Order): Promise<void> {
    // Qui si integrerebbe con un servizio di email (e.g., SendGrid, Nodemailer)
    console.log(`Sending order confirmation for Order #${order.orderNumber} to user ${order.userId || 'guest'}.`);
    // Esempio:
    // await emailService.send({
    //   to: user.email,
    //   subject: `Your Order #${order.orderNumber} Confirmation`,
    //   html: `Thank you for your order! Your order number is ${order.orderNumber}.`,
    // });
  }
}
```

---

## 4. src/server/trpc/routers/cart.ts

```typescript
import { z } from 'zod';
import { publicProcedure, protectedProcedure, router } from '../trpc'; // Assumi che questi siano definiti
import { CartService } from '../services/cart-service';
import { AddItemInputSchema, UpdateItemQuantityInputSchema, ApplyCouponInputSchema } from '../../lib/validations/cart'; // Importa gli schemi Zod

const cartService = new CartService();

export const cartRouter = router({
  /**
   * Recupera il carrello dell'utente autenticato o un carrello guest.
   * Se l'utente è autenticato, cerca il carrello per userId.
   * Se non autenticato, cerca per sessionId.
   * Se non esiste, ne crea uno nuovo.
   */
  getCart: publicProcedure
    .query(async ({ ctx }) => {
      const userId = ctx.session?.user?.id;
      const sessionId = ctx.sessionId; // Assumi che sessionId sia disponibile nel contesto per i guest

      if (!userId && !sessionId) {
        throw new Error('Authentication required or session ID missing.');
      }

      // Se l'utente è autenticato e ha un carrello guest, uniscili
      if (userId && sessionId) {
        const guestCart = await cartService.getCart(sessionId); // Cerca il carrello guest per sessionId
        if (guestCart && guestCart.userId === null) { // Assicurati che sia un vero carrello guest
          await cartService.mergeGuestCart(sessionId, userId);
          // Dopo l'unione, il carrello guest è stato eliminato e il carrello utente è aggiornato.
          // Quindi recuperiamo il carrello utente.
          return cartService.getUserCart(userId);
        }
      }
      
      // Altrimenti, recupera o crea il carrello normalmente
      return cartService.getOrCreateCart(userId, sessionId);
    }),

  /**
   * Aggiunge un item al carrello o aggiorna la quantità se già presente.
   */
  addItem: publicProcedure
    .input(AddItemInputSchema)
    .mutation(async ({ ctx, input }) => {
      const userId = ctx.session?.user?.id;
      const sessionId = ctx.sessionId;

      if (!userId && !sessionId) {
        throw new Error('Authentication required or session ID missing.');
      }

      const cart = await cartService.getOrCreateCart(userId, sessionId);
      await cartService.addItem(cart.id, input);
      return cartService.getCart(cart.id); // Restituisce il carrello aggiornato
    }),

  /**
   * Aggiorna la quantità di un item specifico nel carrello.
   */
  updateItemQuantity: publicProcedure
    .input(UpdateItemQuantityInputSchema)
    .mutation(async ({ ctx, input }) => {
      const userId = ctx.session?.user?.id;
      const sessionId = ctx.sessionId;

      if (!userId && !sessionId) {
        throw new Error('Authentication required or session ID missing.');
      }

      const cart = await cartService.getOrCreateCart(userId, sessionId);
      await cartService.updateItemQuantity(cart.id, input.itemId, input.quantity);
      return cartService.getCart(cart.id); // Restituisce il carrello aggiornato
    }),

  /**
   * Rimuove un item specifico dal carrello.
   */
  removeItem: publicProcedure
    .input(z.object({ itemId: z.string() }))
    .mutation(async ({ ctx, input }) => {
      const userId = ctx.session?.user?.id;
      const sessionId = ctx.sessionId;

      if (!userId && !sessionId) {
        throw new Error('Authentication required or session ID missing.');
      }

      const cart = await cartService.getOrCreateCart(userId, sessionId);
      await cartService.removeItem(cart.id, input.itemId);
      return cartService.getCart(cart.id); // Restituisce il carrello aggiornato
    }),

  /**
   * Svuota completamente il carrello.
   */
  clearCart: publicProcedure
    .mutation(async ({ ctx }) => {
      const userId = ctx.session?.user?.id;
      const sessionId = ctx.sessionId;

      if (!userId && !sessionId) {
        throw new Error('Authentication required or session ID missing.');
      }

      const cart = await cartService.getOrCreateCart(userId, sessionId);
      await cartService.clearCart(cart.id);
      return cartService.getCart(cart.id); // Restituisce il carrello svuotato
    }),

  /**
   * Applica un coupon al carrello.
   */
  applyCoupon: publicProcedure
    .input(ApplyCouponInputSchema)
    .mutation(async ({ ctx, input }) => {
      const userId = ctx.session?.user?.id;
      const sessionId = ctx.sessionId;

      if (!userId && !sessionId) {
        throw new Error('Authentication required or session ID missing.');
      }

      const cart = await cartService.getOrCreateCart(userId, sessionId);
      const result = await cartService.applyCoupon(cart.id, input.code);
      return result.cart; // Restituisce il carrello aggiornato con lo sconto
    }),

  /**
   * Rimuove un coupon dal carrello.
   */
  removeCoupon: publicProcedure
    .mutation(async ({ ctx }) => {
      const userId = ctx.session?.user?.id;
      const sessionId = ctx.sessionId;

      if (!userId && !sessionId) {
        throw new Error('Authentication required or session ID missing.');
      }

      const cart = await cartService.getOrCreateCart(userId, sessionId);
      return cartService.removeCoupon(cart.id); // Restituisce il carrello aggiornato
    }),

  /**
   * Valida lo stock degli item nel carrello.
   */
  validateStock: publicProcedure
    .query(async ({ ctx }) => {
      const userId = ctx.session?.user?.id;
      const sessionId = ctx.sessionId;

      if (!userId && !sessionId) {
        throw new Error('Authentication required or session ID missing.');
      }

      const cart = await cartService.getOrCreateCart(userId, sessionId);
      return cartService.validateStock(cart.id);
    }),
});

// Esempio di contesto tRPC (da adattare alla tua implementazione)
// interface Context {
//   session: { user: { id: string } } | null;
//   sessionId: string | null;
//   // ... altre proprietà del contesto
// }
```

---

## 5. src/server/trpc/routers/checkout.ts

```typescript
import { z } from 'zod';
import { publicProcedure, protectedProcedure, router } from '../trpc'; // Assumi che questi siano definiti
import { CheckoutService } from '../services/checkout-service';
import {
  AddressInputSchema,
  SetShippingMethodInputSchema,
  SetPaymentMethodInputSchema,
} from '../../lib/validations/cart'; // Importa gli schemi Zod
import { CheckoutStep } from '@prisma/client';

const checkoutService = new CheckoutService();

export const checkoutRouter = router({
  /**
   * Crea o recupera una sessione di checkout per il carrello corrente.
   */
  createCheckoutSession: publicProcedure
    .mutation(async ({ ctx }) => {
      const userId = ctx.session?.user?.id;
      const sessionId = ctx.sessionId;

      if (!userId && !sessionId) {
        throw new Error('Authentication required or session ID missing.');
      }

      // Qui dovresti recuperare l'ID del carrello dell'utente/sessione
      // Questo richiede un'interazione con il CartService per ottenere il cartId
      const cart = await checkoutService['cartService'].getOrCreateCart(userId, sessionId); // Accedi a cartService tramite la proprietà privata
      return checkoutService.createCheckoutSession(cart.id);
    }),

  /**
   * Recupera una sessione di checkout esistente.
   */
  getCheckoutSession: publicProcedure
    .input(z.object({ sessionId: z.string() }))
    .query(async ({ input }) => {
      return checkoutService.getCheckoutSession(input.sessionId);
    }),

  /**
   * Imposta l'indirizzo di spedizione per la sessione di checkout.
   */
  setShippingAddress: publicProcedure
    .input(z.object({
      sessionId: z.string(),
      address: AddressInputSchema,
    }))
    .mutation(async ({ input }) => {
      return checkoutService.setShippingAddress(input.sessionId, input.address);
    }),

  /**
   * Imposta l'indirizzo di fatturazione per la sessione di checkout.
   */
  setBillingAddress: publicProcedure
    .input(z.object({
      sessionId: z.string(),
      address: AddressInputSchema,
      sameAsShipping: z.boolean().default(false),
    }))
    .mutation(async ({ input }) => {
      return checkoutService.setBillingAddress(input.sessionId, input.address, input.sameAsShipping);
    }),

  /**
   * Recupera le opzioni di spedizione disponibili per la sessione di checkout.
   */
  getShippingOptions: publicProcedure
    .input(z.object({ sessionId: z.string() }))
    .query(async ({ input }) => {
      return checkoutService.getShippingOptions(input.sessionId);
    }),

  /**
   * Imposta il metodo di spedizione per la sessione di checkout.
   */
  setShippingMethod: publicProcedure
    .input(SetShippingMethodInputSchema)
    .mutation(async ({ input }) => {
      return checkoutService.setShippingMethod(input.sessionId, input.methodId);
    }),

  /**
   * Imposta il metodo di pagamento per la sessione di checkout.
   */
  setPaymentMethod: publicProcedure
    .input(SetPaymentMethodInputSchema)
    .mutation(async ({ input }) => {
      return checkoutService.setPaymentMethod(input.sessionId, input.paymentMethodId);
    }),

  /**
   * Valida lo stato attuale della sessione di checkout.
   */
  validateCheckout: publicProcedure
    .input(z.object({ sessionId: z.string() }))
    .query(async ({ input }) => {
      return checkoutService.validateCheckout(input.sessionId);
    }),

  /**
   * Processa il checkout, creando l'ordine e gestendo il pagamento.
   */
  processCheckout: publicProcedure
    .input(z.object({ sessionId: z.string() }))
    .mutation(async ({ input }) => {
      return checkoutService.processCheckout(input.sessionId);
    }),
});

// Esempio di contesto tRPC (da adattare alla tua implementazione)
// interface Context {
//   session: { user: { id: string } } | null;
//   sessionId: string | null;
//   // ... altre proprietà del contesto
// }
```

---

## 6. src/lib/validations/cart.ts

```typescript
import { z } from 'zod';

// Schema per l'aggiunta di un item al carrello
export const AddItemInputSchema = z.object({
  productId: z.string().min(1, 'Product ID is required.'),
  variantId: z.string().optional(),
  quantity: z.number().int().min(1, 'Quantity must be at least 1.'),
});

// Schema per l'aggiornamento della quantità di un item nel carrello
export const UpdateItemQuantityInputSchema = z.object({
  itemId: z.string().min(1, 'Item ID is required.'),
  quantity: z.number().int().min(0, 'Quantity cannot be negative. Use 0 to remove.'),
});

// Schema per l'applicazione di un coupon
export const ApplyCouponInputSchema = z.object({
  code: z.string().min(1, 'Coupon code is required.'),
});

// Schema per un indirizzo generico
export const AddressInputSchema = z.object({
  firstName: z.string().min(1, 'First name is required.'),
  lastName: z.string().min(1, 'Last name is required.'),
  company: z.string().optional(),
  address1: z.string().min(1, 'Address line 1 is required.'),
  address2: z.string().optional(),
  city: z.string().min(1, 'City is required.'),
  state: z.string().optional(), // Potrebbe essere richiesto per alcuni paesi
  zip: z.string().min(1, 'ZIP code is required.'),
  country: z.string().min(1, 'Country is required.'),
  phone: z.string().optional(), // Potrebbe essere richiesto
});

// Schema per l'impostazione del metodo di spedizione
export const SetShippingMethodInputSchema = z.object({
  sessionId: z.string().min(1, 'Session ID is required.'),
  methodId: z.string().min(1, 'Shipping method ID is required.'),
});

// Schema per l'impostazione del metodo di pagamento
export const SetPaymentMethodInputSchema = z.object({
  sessionId: z.string().min(1, 'Session ID is required.'),
  paymentMethodId: z.string().min(1, 'Payment method ID is required.'),
});

// Schema per la creazione di una sessione di checkout (se necessario, ma spesso implicito)
export const CreateCheckoutSessionInputSchema = z.object({
  cartId: z.string().min(1, 'Cart ID is required.'),
});

// Schema per l'aggiornamento generico di una sessione di checkout
// Non è esposto direttamente via tRPC ma usato internamente dal servizio
export const UpdateCheckoutInputSchema = z.object({
  currentStep: z.enum(['SHIPPING_ADDRESS', 'BILLING_ADDRESS', 'SHIPPING_METHOD', 'PAYMENT', 'REVIEW', 'COMPLETED']).optional(),
  shippingAddressId: z.string().optional(),
  billingAddressId: z.string().optional(),
  shippingMethodId: z.string().optional(),
  shippingMethodName: z.string().optional(),
  shippingMethodPrice: z.number().optional(), // Decimal in Prisma, number in Zod per input
  paymentMethodId: z.string().optional(),
  paymentIntentId: z.string().optional(),
  subtotal: z.number().optional(),
  discount: z.number().optional(),
  tax: z.number().optional(),
  shipping: z.number().optional(),
  total: z.number().optional(),
});
```

---

## 7. src/hooks/use-cart.ts

```typescript
import { useState, useEffect, useMemo, useCallback } from 'react';
import { trpc } from '../utils/trpc'; // Assumi che trpc sia configurato
import { Cart, CartItem } from '@prisma/client'; // Importa i tipi Prisma
import { Decimal } from 'decimal.js'; // Per gestire i Decimal

interface UseCartResult {
  cart: Cart | undefined;
  items: (CartItem & { product: { name: string; slug: string }; variant: { name: string } | null })[];
  itemCount: number;
  subtotal: Decimal;
  discount: Decimal;
  tax: Decimal;
  shipping: Decimal;
  total: Decimal;
  isLoading: boolean;
  isAdding: boolean;
  isUpdating: boolean;
  isRemoving: boolean;
  isClearing: boolean;
  isApplyingCoupon: boolean;
  addItem: (productId: string, quantity: number, variantId?: string) => Promise<void>;
  updateQuantity: (itemId: string, quantity: number) => Promise<void>;
  removeItem: (itemId: string) => Promise<void>;
  clearCart: () => Promise<void>;
  applyCoupon: (code: string) => Promise<void>;
  removeCoupon: () => Promise<void>;
  error: Error | null;
}

export function useCart(): UseCartResult {
  const [error, setError] = useState<Error | null>(null);

  // Query per ottenere il carrello
  const { data: cart, isLoading, refetch } = trpc.cart.getCart.useQuery(undefined, {
    staleTime: 5 * 60 * 1000, // 5 minuti
    onError: (err) => setError(new Error(err.message)),
  });

  // Mutazioni per le operazioni sul carrello
  const addItemMutation = trpc.cart.addItem.useMutation({
    onSuccess: () => refetch(),
    onError: (err) => setError(new Error(err.message)),
  });
  const updateItemQuantityMutation = trpc.cart.updateItemQuantity.useMutation({
    onSuccess: () => refetch(),
    onError: (err) => setError(new Error(err.message)),
  });
  const removeItemMutation = trpc.cart.removeItem.useMutation({
    onSuccess: () => refetch(),
    onError: (err) => setError(new Error(err.message)),
  });
  const clearCartMutation = trpc.cart.clearCart.useMutation({
    onSuccess: () => refetch(),
    onError: (err) => setError(new Error(err.message)),
  });
  const applyCouponMutation = trpc.cart.applyCoupon.useMutation({
    onSuccess: () => refetch(),
    onError: (err) => setError(new Error(err.message)),
  });
  const removeCouponMutation = trpc.cart.removeCoupon.useMutation({
    onSuccess: () => refetch(),
    onError: (err) => setError(new Error(err.message)),
  });

  // Dati derivati dal carrello
  const items = useMemo(() => {
    // Assicurati che `items` sia sempre un array, anche se `cart` è undefined
    return (cart?.items || []) as (CartItem & { product: { name: string; slug: string }; variant: { name: string } | null })[];
  }, [cart]);

  const itemCount = useMemo(() => {
    return items.reduce((count, item) => count + item.quantity, 0);
  }, [items]);

  const subtotal = useMemo(() => new Decimal(cart?.subtotal || 0), [cart?.subtotal]);
  const discount = useMemo(() => new Decimal(cart?.discount || 0), [cart?.discount]);
  const tax = useMemo(() => new Decimal(cart?.tax || 0), [cart?.tax]);
  const shipping = useMemo(() => new Decimal(cart?.shipping || 0), [cart?.shipping]);
  const total = useMemo(() => new Decimal(cart?.total || 0), [cart?.total]);

  // Funzioni per interagire con il carrello
  const addItem = useCallback(async (productId: string, quantity: number, variantId?: string) => {
    setError(null);
    await addItemMutation.mutateAsync({ productId, quantity, variantId });
  }, [addItemMutation]);

  const updateQuantity = useCallback(async (itemId: string, quantity: number) => {
    setError(null);
    await updateItemQuantityMutation.mutateAsync({ itemId, quantity });
  }, [updateItemQuantityMutation]);

  const removeItem = useCallback(async (itemId: string) => {
    setError(null);
    await removeItemMutation.mutateAsync({ itemId });
  }, [removeItemMutation]);

  const clearCart = useCallback(async () => {
    setError(null);
    await clearCartMutation.mutateAsync();
  }, [clearCartMutation]);

  const applyCoupon = useCallback(async (code: string) => {
    setError(null);
    await applyCouponMutation.mutateAsync({ code });
  }, [applyCouponMutation]);

  const removeCoupon = useCallback(async () => {
    setError(null);
    await removeCouponMutation.mutateAsync();
  }, [removeCouponMutation]);

  return {
    cart,
    items,
    itemCount,
    subtotal,
    discount,
    tax,
    shipping,
    total,
    isLoading,
    isAdding: addItemMutation.isLoading,
    isUpdating: updateItemQuantityMutation.isLoading,
    isRemoving: removeItemMutation.isLoading,
    isClearing: clearCartMutation.isLoading,
    isApplyingCoupon: applyCouponMutation.isLoading || removeCouponMutation.isLoading,
    addItem,
    updateQuantity,
    removeItem,
    clearCart,
    applyCoupon,
    removeCoupon,
    error,
  };
}
```

---

## 8. src/hooks/use-checkout.ts

```typescript
import { useState, useEffect, useCallback, useMemo } from 'react';
import { trpc } from '../utils/trpc'; // Assumi che trpc sia configurato
import { CheckoutSession, CheckoutStep, Address } from '@prisma/client';
import { AddressInput, ShippingMethod } from '../server/services/checkout-service'; // Importa i tipi
import { useRouter } from "next/navigation"; // O 'next/navigation' per App Router

interface UseCheckoutResult {
  session: CheckoutSession | undefined;
  currentStep: CheckoutStep;
  isLoading: boolean;
  isProcessing: boolean;
  error: Error | null;
  shippingOptions: ShippingMethod[];
  setShippingAddress: (address: AddressInput) => Promise<void>;
  setBillingAddress: (address: AddressInput, sameAsShipping?: boolean) => Promise<void>;
  setShippingMethod: (methodId: string) => Promise<void>;
  setPaymentMethod: (paymentMethodId: string) => Promise<void>;
  goToStep: (step: CheckoutStep) => void;
  processCheckout: () => Promise<void>;
  refetchSession: () => Promise<any>;
}

export function useCheckout(): UseCheckoutResult {
  const router = useRouter();
  const [checkoutSessionId, setCheckoutSessionId] = useState<string | undefined>(undefined);
  const [error, setError] = useState<Error | null>(null);
  const [shippingOptions, setShippingOptions] = useState<ShippingMethod[]>([]);

  // Query per creare/ottenere la sessione di checkout
  const createSessionMutation = trpc.checkout.createCheckoutSession.useMutation({
    onSuccess: (data) => {
      setCheckoutSessionId(data.id);
      // Salva l'ID della sessione in localStorage o in un cookie per persistenza
      localStorage.setItem('checkoutSessionId', data.id);
    },
    onError: (err) => setError(new Error(err.message)),
  });

  const { data: session, isLoading, refetch: refetchSession } = trpc.checkout.getCheckoutSession.useQuery(
    { sessionId: checkoutSessionId! },
    {
      enabled: !!checkoutSessionId, // Esegui la query solo se abbiamo un sessionId
      staleTime: 5 * 60 * 1000, // 5 minuti
      onError: (err) => setError(new Error(err.message)),
    }
  );

  // Effetto per inizializzare la sessione o recuperarla da localStorage
  useEffect(() => {
    const storedSessionId = localStorage.getItem('checkoutSessionId');
    if (storedSessionId) {
      setCheckoutSessionId(storedSessionId);
    } else {
      // Se non c'è un ID salvato, crea una nuova sessione
      createSessionMutation.mutate();
    }
  }, []); // Esegui solo al mount

  // Effetto per recuperare le opzioni di spedizione quando l'indirizzo è impostato
  useEffect(() => {
    if (session?.shippingAddressId && session.currentStep === CheckoutStep.SHIPPING_METHOD) {
      trpc.checkout.getShippingOptions.query({ sessionId: session.id })
        .then((options) => setShippingOptions(options as ShippingMethod[])) // Cast esplicito
        .catch((err) => setError(new Error(err.message)));
    }
  }, [session?.shippingAddressId, session?.currentStep, session?.id]);

  const currentStep = useMemo(() => session?.currentStep || CheckoutStep.SHIPPING_ADDRESS, [session?.currentStep]);

  // Mutazioni per i passaggi del checkout
  const setShippingAddressMutation = trpc.checkout.setShippingAddress.useMutation({
    onSuccess: () => refetchSession(),
    onError: (err) => setError(new Error(err.message)),
  });
  const setBillingAddressMutation = trpc.checkout.setBillingAddress.useMutation({
    onSuccess: () => refetchSession(),
    onError: (err) => setError(new Error(err.message)),
  });
  const setShippingMethodMutation = trpc.checkout.setShippingMethod.useMutation({
    onSuccess: () => refetchSession(),
    onError: (err) => setError(new Error(err.message)),
  });
  const setPaymentMethodMutation = trpc.checkout.setPaymentMethod.useMutation({
    onSuccess: () => refetchSession(),
    onError: (err) => setError(new Error(err.message)),
  });
  const processCheckoutMutation = trpc.checkout.processCheckout.useMutation({
    onSuccess: (order) => {
      localStorage.removeItem('checkoutSessionId'); // Pulisci la sessione dopo il successo
      router.push(`/checkout/success?orderId=${order.id}`);
    },
    onError: (err) => setError(new Error(err.message)),
  });

  const setShippingAddress = useCallback(async (address: AddressInput) => {
    setError(null);
    if (checkoutSessionId) {
      await setShippingAddressMutation.mutateAsync({ sessionId: checkoutSessionId, address });
    }
  }, [checkoutSessionId, setShippingAddressMutation]);

  const setBillingAddress = useCallback(async (address: AddressInput, sameAsShipping: boolean = false) => {
    setError(null);
    if (checkoutSessionId) {
      await setBillingAddressMutation.mutateAsync({ sessionId: checkoutSessionId, address, sameAsShipping });
    }
  }, [checkoutSessionId, setBillingAddressMutation]);

  const setShippingMethod = useCallback(async (methodId: string) => {
    setError(null);
    if (checkoutSessionId) {
      await setShippingMethodMutation.mutateAsync({ sessionId: checkoutSessionId, methodId });
    }
  }, [checkoutSessionId, setShippingMethodMutation]);

  const setPaymentMethod = useCallback(async (paymentMethodId: string) => {
    setError(null);
    if (checkoutSessionId) {
      await setPaymentMethodMutation.mutateAsync({ sessionId: checkoutSessionId, paymentMethodId });
    }
  }, [checkoutSessionId, setPaymentMethodMutation]);

  const goToStep = useCallback((step: CheckoutStep) => {
    // Implementa la logica per navigare tra i passi,
    // potenzialmente aggiornando la sessione nel backend se necessario
    // Per ora, solo un placeholder
    console.log(`Navigating to step: ${step}`);
  }, []);

  const processCheckout = useCallback(async () => {
    setError(null);
    if (checkoutSessionId) {
      await processCheckoutMutation.mutateAsync({ sessionId: checkoutSessionId });
    }
  }, [checkoutSessionId, processCheckoutMutation]);

  return {
    session,
    currentStep,
    isLoading: isLoading || createSessionMutation.isLoading,
    isProcessing: processCheckoutMutation.isLoading,
    error,
    shippingOptions,
    setShippingAddress,
    setBillingAddress,
    setShippingMethod,
    setPaymentMethod,
    goToStep,
    processCheckout,
    refetchSession,
  };
}
```

---

## 9. src/components/cart/cart-drawer.tsx

```typescript
import React from 'react';
import { useCart } from '../../hooks/use-cart';
import { CartItemCard } from './cart-item';
import { CartSummary } from './cart-summary';
import Link from 'next/link';

// Questo è un componente di esempio per un drawer.
// In un'applicazione reale, useresti una libreria UI come Shadcn UI, Headless UI, Material UI, ecc.
// per gestire l'apertura/chiusura e l'animazione del drawer.
interface CartDrawerProps {
  isOpen: boolean;
  onClose: () => void;
}

export function CartDrawer({ isOpen, onClose }: CartDrawerProps) {
  const { items, itemCount, isLoading, error } = useCart();

  if (!isOpen) return null;

  return (
    <div
      className="fixed inset-0 z-50 overflow-hidden"
      aria-labelledby="cart-drawer-title"
      role="dialog"
      aria-modal="true"
    >
      <div className="absolute inset-0 bg-gray-500 bg-opacity-75 transition-opacity" onClick={onClose}></div>

      <div className="fixed inset-y-0 right-0 max-w-full flex">
        <div className="w-screen max-w-md">
          <div className="h-full flex flex-col bg-white shadow-xl overflow-y-scroll">
            <div className="flex-1 py-6 px-4 sm:px-6">
              <div className="flex items-start justify-between">
                <h2 className="text-lg font-medium text-gray-900" id="cart-drawer-title">
                  Shopping Cart ({itemCount})
                </h2>
                <div className="ml-3 flex h-7 items-center">
                  <button
                    type="button"
                    className="-m-2 p-2 text-gray-400 hover:text-gray-500"
                    onClick={onClose}
                  >
                    <span className="sr-only">Close panel</span>
                    {/* Icona X */}
                    <svg className="h-6 w-6" fill="none" viewBox="0 0 24 24" strokeWidth="1.5" stroke="currentColor" aria-hidden="true">
                      <path strokeLinecap="round" strokeLinejoin="round" d="M6 18L18 6M6 6l12 12" />
                    </svg>
                  </button>
                </div>
              </div>

              <div className="mt-8">
                {isLoading ? (
                  <p className="text-center text-gray-500">Loading cart...</p>
                ) : error ? (
                  <p className="text-center text-red-500">Error loading cart: {error.message}</p>
                ) : items.length === 0 ? (
                  <p className="text-center text-gray-500">Your cart is empty.</p>
                ) : (
                  <ul role="list" className="-my-6 divide-y divide-gray-200">
                    {items.map((item) => (
                      <CartItemCard key={item.id} item={item} />
                    ))}
                  </ul>
                )}
              </div>
            </div>

            <div className="border-t border-gray-200 py-6 px-4 sm:px-6">
              <CartSummary />

              <div className="mt-6">
                <Link
                  href="/checkout"
                  className="flex items-center justify-center rounded-md border border-transparent bg-indigo-600 px-6 py-3 text-base font-medium text-white shadow-sm hover:bg-indigo-700"
                  onClick={onClose}
                >
                  Checkout
                </Link>
              </div>
              <div className="mt-6 flex justify-center text-center text-sm text-gray-500">
                <p>
                  or{' '}
                  <button
                    type="button"
                    className="font-medium text-indigo-600 hover:text-indigo-500"
                    onClick={onClose}
                  >
                    Continue Shopping
                    <span aria-hidden="true"> &rarr;</span>
                  </button>
                </p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  );
}
```

---

## 10. src/components/cart/cart-item.tsx

```typescript
import React from 'react';
import Image from 'next/image';
import { useCart } from '../../hooks/use-cart';
import { CartItem } from '@prisma/client';
import { Decimal } from 'decimal.js';

// Estendi il tipo CartItem per includere le relazioni necessarie
type CartItemWithProductAndVariant = CartItem & {
  product: { name: string; slug: string; imageUrl?: string };
  variant: { name: string } | null;
};

interface CartItemCardProps {
  item: CartItemWithProductAndVariant;
}

export function CartItemCard({ item }: CartItemCardProps) {
  const { updateQuantity, removeItem, isUpdating, isRemoving } = useCart();

  const handleQuantityChange = async (newQuantity: number) => {
    if (newQuantity < 0) return; // Previene quantità negative
    await updateQuantity(item.id, newQuantity);
  };

  const handleRemoveItem = async () => {
    await removeItem(item.id);
  };

  const itemPrice = new Decimal(item.price);
  const itemTotal = itemPrice.times(item.quantity);

  return (
    <li className="flex py-6">
      <div className="h-24 w-24 flex-shrink-0 overflow-hidden rounded-md border border-gray-200">
        <Image
          src={item.product.imageUrl || '/placeholder-product.png'} // Immagine placeholder
          alt={item.product.name}
          width={96}
          height={96}
          className="h-full w-full object-cover object-center"
        />
      </div>

      <div className="ml-4 flex flex-1 flex-col">
        <div>
          <div className="flex justify-between text-base font-medium text-gray-900">
            <h3>
              <a href={`/products/${item.product.slug}`}>{item.product.name}</a>
            </h3>
            <p className="ml-4">{itemTotal.toFixed(2)} €</p>
          </div>
          <p className="mt-1 text-sm text-gray-500">
            {item.variant ? item.variant.name : 'Standard'}
          </p>
        </div>
        <div className="flex flex-1 items-end justify-between text-sm">
          <div className="flex items-center">
            <label htmlFor={`quantity-${item.id}`} className="sr-only">
              Quantity
            </label>
            <button
              onClick={() => handleQuantityChange(item.quantity - 1)}
              disabled={isUpdating || item.quantity <= 1}
              className="px-2 py-1 border border-gray-300 rounded-l-md text-gray-600 hover:bg-gray-100 disabled:opacity-50"
            >
              -
            </button>
            <input
              id={`quantity-${item.id}`}
              type="number"
              value={item.quantity}
              onChange={(e) => handleQuantityChange(parseInt(e.target.value))}
              disabled={isUpdating}
              className="w-12 text-center border-t border-b border-gray-300 py-1 text-gray-900 focus:outline-none"
              min="1"
            />
            <button
              onClick={() => handleQuantityChange(item.quantity + 1)}
              disabled={isUpdating}
              className="px-2 py-1 border border-gray-300 rounded-r-md text-gray-600 hover:bg-gray-100 disabled:opacity-50"
            >
              +
            </button>
          </div>

          <div className="flex">
            <button
              type="button"
              onClick={handleRemoveItem}
              disabled={isRemoving}
              className="font-medium text-indigo-600 hover:text-indigo-500 disabled:opacity-50"
            >
              {isRemoving ? 'Removing...' : 'Remove'}
            </button>
          </div>
        </div>
      </div>
    </li>
  );
}
```

---

## 11. src/components/cart/cart-summary.tsx

```typescript
import React, { useState } from 'react';
import { useCart } from '../../hooks/use-cart';

export function CartSummary() {
  const { subtotal, discount, tax, shipping, total, applyCoupon, removeCoupon, cart, isApplyingCoupon, error } = useCart();
  const [couponCode, setCouponCode] = useState(cart?.coupon?.code || '');
  const [couponError, setCouponError] = useState<string | null>(null);

  const handleApplyCoupon = async () => {
    setCouponError(null);
    try {
      await applyCoupon(couponCode);
      setCouponCode(''); // Pulisci l'input dopo l'applicazione
    } catch (err: any) {
      setCouponError(err.message || 'Failed to apply coupon.');
    }
  };

  const handleRemoveCoupon = async () => {
    setCouponError(null);
    try {
      await removeCoupon();
      setCouponCode(''); // Pulisci l'input dopo la rimozione
    } catch (err: any) {
      setCouponError(err.message || 'Failed to remove coupon.');
    }
  };

  return (
    <div className="space-y-4">
      <div className="flex justify-between text-base font-medium text-gray-900">
        <p>Subtotal</p>
        <p>{subtotal.toFixed(2)} €</p>
      </div>
      {discount.greaterThan(0) && (
        <div className="flex justify-between text-sm text-green-600">
          <p>Discount</p>
          <p>-{discount.toFixed(2)} €</p>
        </div>
      )}
      <div className="flex justify-between text-sm text-gray-500">
        <p>Shipping</p>
        <p>{shipping.toFixed(2)} €</p>
      </div>
      <div className="flex justify-between text-sm text-gray-500">
        <p>Tax</p>
        <p>{tax.toFixed(2)} €</p>
      </div>
      <div className="flex justify-between text-lg font-bold text-gray-900 border-t pt-4 mt-4">
        <p>Order Total</p>
        <p>{total.toFixed(2)} €</p>
      </div>

      <div className="mt-6">
        <label htmlFor="coupon-code" className="block text-sm font-medium text-gray-700">
          Have a coupon?
        </label>
        <div className="mt-1 flex space-x-2">
          <input
            type="text"
            id="coupon-code"
            name="coupon-code"
            value={couponCode}
            onChange={(e) => setCouponCode(e.target.value)}
            className="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm"
            placeholder="Enter coupon code"
            disabled={isApplyingCoupon}
          />
          {cart?.couponId ? (
            <button
              type="button"
              onClick={handleRemoveCoupon}
              disabled={isApplyingCoupon}
              className="whitespace-nowrap rounded-md border border-transparent bg-red-600 px-4 py-2 text-sm font-medium text-white shadow-sm hover:bg-red-700 disabled:opacity-50"
            >
              {isApplyingCoupon ? 'Removing...' : 'Remove'}
            </button>
          ) : (
            <button
              type="button"
              onClick={handleApplyCoupon}
              disabled={isApplyingCoupon || !couponCode.trim()}
              className="whitespace-nowrap rounded-md border border-transparent bg-indigo-600 px-4 py-2 text-sm font-medium text-white shadow-sm hover:bg-indigo-700 disabled:opacity-50"
            >
              {isApplyingCoupon ? 'Applying...' : 'Apply'}
            </button>
          )}
        </div>
        {couponError && <p className="mt-2 text-sm text-red-600">{couponError}</p>}
        {cart?.coupon && !couponError && (
          <p className="mt-2 text-sm text-green-600">Coupon "{cart.coupon.code}" applied!</p>
        )}
      </div>
    </div>
  );
}
```

---

## 12. src/components/cart/cart-icon.tsx

```typescript
import React from 'react';
import { useCart } from '../../hooks/use-cart';

interface CartIconProps {
  onClick: () => void;
}

export function CartIcon({ onClick }: CartIconProps) {
  const { itemCount, isLoading } = useCart();

  return (
    <button
      type="button"
      onClick={onClick}
      className="relative p-2 text-gray-400 hover:text-gray-500"
      aria-label="Open shopping cart"
    >
      <span className="sr-only">Items in cart</span>
      {/* Icona carrello SVG */}
      <svg
        className="h-6 w-6"
        fill="none"
        viewBox="0 0 24 24"
        strokeWidth="1.5"
        stroke="currentColor"
        aria-hidden="true"
      >
        <path
          strokeLinecap="round"
          strokeLinejoin="round"
          d="M2.25 3h1.386c.51 0 .955.343 1.023.832l.58 3.492m-1.5 7.5a.75.75 0 11-1.5 0 .75.75 0 011.5 0zM19.5 3h1.386c.51 0 .955.343 1.023.832l.58 3.492m-1.5 7.5a.75.75 0 11-1.5 0 .75.75 0 011.5 0zM6.75 10.5h10.5c.675 0 1.147.638 1.087 1.312l-1.5 9A1.125 1.125 0 0117.023 21H9.487a1.125 1.125 0 01-1.12-1.212l-1.5-9A1.125 1.125 0 016.75 10.5z"
        />
      </svg>

      {itemCount > 0 && !isLoading && (
        <span className="absolute top-0 right-0 inline-flex items-center justify-center px-2 py-1 text-xs font-bold leading-none text-red-100 bg-red-600 rounded-full transform translate-x-1/2 -translate-y-1/2">
          {itemCount}
        </span>
      )}
    </button>
  );
}
```

---

## 13. src/components/checkout/checkout-form.tsx

```typescript
import React, { useState, useEffect } from 'react';
import { useCheckout } from '../../hooks/use-checkout';
import { CheckoutStep } from '@prisma/client';
import { AddressForm } from './address-form';
import { ShippingOptions } from './shipping-options';
import { OrderSummary } from './order-summary';
import { z } from 'zod';
import { AddressInputSchema } from '../../lib/validations/cart';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';

// Schemi per i form specifici del checkout
const ShippingAddressFormSchema = AddressInputSchema;
const BillingAddressFormSchema = AddressInputSchema.extend({
  sameAsShipping: z.boolean().optional(),
});

type ShippingAddressFormData = z.infer<typeof ShippingAddressFormSchema>;
type BillingAddressFormData = z.infer<typeof BillingAddressFormSchema>;

export function CheckoutForm() {
  const {
    session,
    currentStep,
    isLoading,
    isProcessing,
    error,
    setShippingAddress,
    setBillingAddress,
    setShippingMethod,
    setPaymentMethod,
    processCheckout,
    refetchSession,
  } = useCheckout();

  const [paymentMethodId, setPaymentMethodId] = useState<string>(''); // Placeholder per l'ID del metodo di pagamento
  const [stripeElementsReady, setStripeElementsReady] = useState(false); // Per simulare Stripe

  // Form per l'indirizzo di spedizione
  const shippingAddressForm = useForm<ShippingAddressFormData>({
    resolver: zodResolver(ShippingAddressFormSchema),
    defaultValues: session?.shippingAddress || {},
  });

  // Form per l'indirizzo di fatturazione
  const billingAddressForm = useForm<BillingAddressFormData>({
    resolver: zodResolver(BillingAddressFormSchema),
    defaultValues: session?.billingAddress || { sameAsShipping: true },
  });

  // Aggiorna i defaultValues quando la sessione cambia
  useEffect(() => {
    if (session?.shippingAddress) {
      shippingAddressForm.reset(session.shippingAddress);
    }
    if (session?.billingAddress) {
      billingAddressForm.reset({ ...session.billingAddress, sameAsShipping: session.shippingAddressId === session.billingAddressId });
    } else if (session?.shippingAddress && billingAddressForm.getValues('sameAsShipping')) {
      // Se l'utente ha selezionato "same as shipping" e non c'è un billingAddress, usa quello di spedizione
      billingAddressForm.reset({ ...session.shippingAddress, sameAsShipping: true });
    }
  }, [session, shippingAddressForm, billingAddressForm]);

  // Gestione sottomissione indirizzo di spedizione
  const onSubmitShippingAddress = async (data: ShippingAddressFormData) => {
    await setShippingAddress(data);
  };

  // Gestione sottomissione indirizzo di fatturazione
  const onSubmitBillingAddress = async (data: BillingAddressFormData) => {
    const { sameAsShipping, ...addressData } = data;
    await setBillingAddress(addressData, sameAsShipping);
  };

  // Gestione selezione metodo di spedizione
  const handleSelectShippingMethod = async (methodId: string) => {
    await setShippingMethod(methodId);
  };

  // Gestione sottomissione pagamento (placeholder per Stripe)
  const handlePaymentSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!paymentMethodId) {
      setError(new Error('Please select a payment method.'));
      return;
    }
    await setPaymentMethod(paymentMethodId);
  };

  // Gestione del processo di checkout finale
  const handleProcessCheckout = async () => {
    await processCheckout();
  };

  if (isLoading || !session) {
    return <div className="text-center py-10">Loading checkout...</div>;
  }

  if (error) {
    return <div className="text-center py-10 text-red-600">Error: {error.message}</div>;
  }

  // Funzione per navigare tra i passi (per i bottoni "Next" e "Back")
  const navigateToStep = async (step: CheckoutStep) => {
    // Qui potresti voler aggiungere logica di validazione prima di permettere di andare avanti/indietro
    // Per ora, ricarichiamo la sessione per aggiornare lo stato del passo
    await refetchSession();
  };

  const renderStepContent = () => {
    switch (currentStep) {
      case CheckoutStep.SHIPPING_ADDRESS:
        return (
          <AddressForm
            form={shippingAddressForm}
            onSubmit={onSubmitShippingAddress}
            title="Shipping Address"
            buttonText="Continue to Billing"
          />
        );
      case CheckoutStep.BILLING_ADDRESS:
        return (
          <AddressForm
            form={billingAddressForm}
            onSubmit={onSubmitBillingAddress}
            title="Billing Address"
            buttonText="Continue to Shipping Method"
            showSameAsShipping={true}
            sameAsShippingValue={billingAddressForm.watch('sameAsShipping')}
            onSameAsShippingChange={(checked) => {
              billingAddressForm.setValue('sameAsShipping', checked);
              if (checked && session?.shippingAddress) {
                billingAddressForm.reset(session.shippingAddress);
                billingAddressForm.setValue('sameAsShipping', true); // Assicurati che rimanga true
              } else {
                // Pulisci i campi se si deseleziona "same as shipping"
                billingAddressForm.reset({ sameAsShipping: false });
              }
            }}
          />
        );
      case CheckoutStep.SHIPPING_METHOD:
        return (
          <div>
            <h2 className="text-2xl font-bold mb-6">Shipping Method</h2>
            <ShippingOptions
              selectedMethodId={session.shippingMethodId || undefined}
              onSelectMethod={handleSelectShippingMethod}
            />
            <div className="mt-8 flex justify-between">
              <button
                type="button"
                onClick={() => navigateToStep(CheckoutStep.BILLING_ADDRESS)}
                className="rounded-md border border-gray-300 bg-white py-2 px-4 text-sm font-medium text-gray-700 shadow-sm hover:bg-gray-50"
              >
                Back
              </button>
              <button
                type="button"
                onClick={() => navigateToStep(CheckoutStep.PAYMENT)}
                disabled={!session.shippingMethodId}
                className="rounded-md border border-transparent bg-indigo-600 py-2 px-4 text-sm font-medium text-white shadow-sm hover:bg-indigo-700 disabled:opacity-50"
              >
                Continue to Payment
              </button>
            </div>
          </div>
        );
      case CheckoutStep.PAYMENT:
        return (
          <div>
            <h2 className="text-2xl font-bold mb-6">Payment Information</h2>
            <form onSubmit={handlePaymentSubmit} className="space-y-4">
              {/* Placeholder per Stripe Elements */}
              <div className="border p-4 rounded-md bg-gray-50">
                <p className="text-gray-700">
                  {/* Qui si integrerebbe con Stripe Elements o un altro gateway di pagamento */}
                  Payment form goes here (e.g., Card Element from Stripe).
                </p>
                <div className="mt-4">
                  <label htmlFor="paymentMethod" className="block text-sm font-medium text-gray-700">
                    Select Payment Method (Placeholder)
                  </label>
                  <select
                    id="paymentMethod"
                    name="paymentMethod"
                    value={paymentMethodId}
                    onChange={(e) => setPaymentMethodId(e.target.value)}
                    className="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm"
                  >
                    <option value="">Select...</option>
                    <option value="stripe_card">Credit Card (Stripe)</option>
                    <option value="paypal">PayPal</option>
                  </select>
                </div>
              </div>
              <div className="mt-8 flex justify-between">
                <button
                  type="button"
                  onClick={() => navigateToStep(CheckoutStep.SHIPPING_METHOD)}
                  className="rounded-md border border-gray-300 bg-white py-2 px-4 text-sm font-medium text-gray-700 shadow-sm hover:bg-gray-50"
                >
                  Back
                </button>
                <button
                  type="submit"
                  disabled={!paymentMethodId}
                  className="rounded-md border border-transparent bg-indigo-600 py-2 px-4 text-sm font-medium text-white shadow-sm hover:bg-indigo-700 disabled:opacity-50"
                >
                  Continue to Review
                </button>
              </div>
            </form>
          </div>
        );
      case CheckoutStep.REVIEW:
        return (
          <div>
            <h2 className="text-2xl font-bold mb-6">Order Review</h2>
            <OrderSummary session={session} />
            <div className="mt-8 flex justify-between">
              <button
                type="button"
                onClick={() => navigateToStep(CheckoutStep.PAYMENT)}
                className="rounded-md border border-gray-300 bg-white py-2 px-4 text-sm font-medium text-gray-700 shadow-sm hover:bg-gray-50"
              >
                Back
              </button>
              <button
                type="button"
                onClick={handleProcessCheckout}
                disabled={isProcessing}
                className="rounded-md border border-transparent bg-green-600 py-2 px-4 text-sm font-medium text-white shadow-sm hover:bg-green-700 disabled:opacity-50"
              >
                {isProcessing ? 'Processing Order...' : 'Place Order'}
              </button>
            </div>
          </div>
        );
      case CheckoutStep.COMPLETED:
        return (
          <div className="text-center py-10">
            <h2 className="text-2xl font-bold text-green-600">Order Placed Successfully!</h2>
            <p className="mt-4 text-gray-700">Redirecting to order confirmation page...</p>
          </div>
        );
      default:
        return <div className="text-center py-10">Unknown checkout step.</div>;
    }
  };

  return (
    <div className="grid grid-cols-1 lg:grid-cols-3 gap-8">
      <div className="lg:col-span-2 bg-white p-8 rounded-lg shadow-md">
        {renderStepContent()}
      </div>
      <div className="lg:col-span-1 bg-gray-50 p-6 rounded-lg shadow-md h-fit sticky top-8">
        <h3 className="text-xl font-bold mb-4">Your Order</h3>
        <OrderSummary session={session} />
      </div>
    </div>
  );
}
```

---

## 14. src/components/checkout/address-form.tsx

```typescript
import React from 'react';
import { useForm, UseFormReturn } from 'react-hook-form';
import { z } from 'zod';
import { AddressInputSchema } from '../../lib/validations/cart';
import { zodResolver } from '@hookform/resolvers/zod';

// Estendi lo schema per includere l'opzione "sameAsShipping"
const FormSchema = AddressInputSchema.extend({
  sameAsShipping: z.boolean().optional(),
});

type AddressFormData = z.infer<typeof FormSchema>;

interface AddressFormProps {
  form: UseFormReturn<AddressFormData>; // Passa l'istanza del form da useForm
  onSubmit: (data: AddressFormData) => Promise<void>;
  title: string;
  buttonText: string;
  showSameAsShipping?: boolean;
  sameAsShippingValue?: boolean;
  onSameAsShippingChange?: (checked: boolean) => void;
}

export function AddressForm({
  form,
  onSubmit,
  title,
  buttonText,
  showSameAsShipping = false,
  sameAsShippingValue,
  onSameAsShippingChange,
}: AddressFormProps) {
  const { register, handleSubmit, formState: { errors, isSubmitting }, watch } = form;

  const isSameAsShipping = watch('sameAsShipping');

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-6">
      <h2 className="text-2xl font-bold">{title}</h2>

      {showSameAsShipping && (
        <div className="relative flex items-start">
          <div className="flex h-6 items-center">
            <input
              id="sameAsShipping"
              aria-describedby="same-as-shipping-description"
              name="sameAsShipping"
              type="checkbox"
              checked={sameAsShippingValue}
              onChange={(e) => onSameAsShippingChange?.(e.target.checked)}
              className="h-4 w-4 rounded border-gray-300 text-indigo-600 focus:ring-indigo-600"
            />
          </div>
          <div className="ml-3 text-sm leading-6">
            <label htmlFor="sameAsShipping" className="font-medium text-gray-900">
              Same as shipping address
            </label>
          </div>
        </div>
      )}

      {!isSameAsShipping && (
        <>
          <div className="grid grid-cols-1 gap-y-6 sm:grid-cols-2 sm:gap-x-4">
            <div>
              <label htmlFor="firstName" className="block text-sm font-medium text-gray-700">
                First name
              </label>
              <div className="mt-1">
                <input
                  type="text"
                  id="firstName"
                  {...register('firstName')}
                  className="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm"
                />
                {errors.firstName && <p className="mt-1 text-sm text-red-600">{errors.firstName.message}</p>}
              </div>
            </div>

            <div>
              <label htmlFor="lastName" className="block text-sm font-medium text-gray-700">
                Last name
              </label>
              <div className="mt-1">
                <input
                  type="text"
                  id="lastName"
                  {...register('lastName')}
                  className="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm"
                />
                {errors.lastName && <p className="mt-1 text-sm text-red-600">{errors.lastName.message}</p>}
              </div>
            </div>
          </div>

          <div>
            <label htmlFor="company" className="block text-sm font-medium text-gray-700">
              Company (Optional)
            </label>
            <div className="mt-1">
              <input
                type="text"
                id="company"
                {...register('company')}
                className="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm"
              />
            </div>
          </div>

          <div>
            <label htmlFor="address1" className="block text-sm font-medium text-gray-700">
              Address line 1
            </label>
            <div className="mt-1">
              <input
                type="text"
                id="address1"
                {...register('address1')}
                className="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm"
              />
              {errors.address1 && <p className="mt-1 text-sm text-red-600">{errors.address1.message}</p>}
            </div>
          </div>

          <div>
            <label htmlFor="address2" className="block text-sm font-medium text-gray-700">
              Address line 2 (Optional)
            </label>
            <div className="mt-1">
              <input
                type="text"
                id="address2"
                {...register('address2')}
                className="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm"
              />
            </div>
          </div>

          <div className="grid grid-cols-1 gap-y-6 sm:grid-cols-3 sm:gap-x-4">
            <div>
              <label htmlFor="city" className="block text-sm font-medium text-gray-700">
                City
              </label>
              <div className="mt-1">
                <input
                  type="text"
                  id="city"
                  {...register('city')}
                  className="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm"
                />
                {errors.city && <p className="mt-1 text-sm text-red-600">{errors.city.message}</p>}
              </div>
            </div>

            <div>
              <label htmlFor="state" className="block text-sm font-medium text-gray-700">
                State / Province (Optional)
              </label>
              <div className="mt-1">
                <input
                  type="text"
                  id="state"
                  {...register('state')}
                  className="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm"
                />
              </div>
            </div>

            <div>
              <label htmlFor="zip" className="block text-sm font-medium text-gray-700">
                ZIP / Postal code
              </label>
              <div className="mt-1">
                <input
                  type="text"
                  id="zip"
                  {...register('zip')}
                  className="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm"
                />
                {errors.zip && <p className="mt-1 text-sm text-red-600">{errors.zip.message}</p>}
              </div>
            </div>
          </div>

          <div>
            <label htmlFor="country" className="block text-sm font-medium text-gray-700">
              Country
            </label>
            <div className="mt-1">
              <input
                type="text"
                id="country"
                {...register('country')}
                className="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm"
              />
              {errors.country && <p className="mt-1 text-sm text-red-600">{errors.country.message}</p>}
            </div>
          </div>

          <div>
            <label htmlFor="phone" className="block text-sm font-medium text-gray-700">
              Phone (Optional)
            </label>
            <div className="mt-1">
              <input
                type="text"
                id="phone"
                {...register('phone')}
                className="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm"
              />
            </div>
          </div>
        </>
      )}

      <div className="mt-6 flex justify-end">
        <button
          type="submit"
          disabled={isSubmitting}
          className="rounded-md border border-transparent bg-indigo-600 py-2 px-4 text-sm font-medium text-white shadow-sm hover:bg-indigo-700 disabled:opacity-50"
        >
          {isSubmitting ? 'Saving...' : buttonText}
        </button>
      </div>
    </form>
  );
}
```

---

## 15. src/components/checkout/shipping-options.tsx

```typescript
import React, { useEffect } from 'react';
import { useCheckout } from '../../hooks/use-checkout';
import { ShippingMethod } from '../../server/services/checkout-service'; // Importa il tipo

interface ShippingOptionsProps {
  selectedMethodId?: string;
  onSelectMethod: (methodId: string) => void;
}

export function ShippingOptions({ selectedMethodId, onSelectMethod }: ShippingOptionsProps) {
  const { shippingOptions, isLoading, error } = useCheckout();

  useEffect(() => {
    // Se c'è solo un'opzione e nessuna è selezionata, selezionala automaticamente
    if (shippingOptions.length === 1 && !selectedMethodId) {
      onSelectMethod(shippingOptions[0].id);
    }
  }, [shippingOptions, selectedMethodId, onSelectMethod]);

  if (isLoading) {
    return <div className="text-center py-4 text-gray-500">Loading shipping options...</div>;

---
_Modello: gemini-2.5-flash (Google AI Studio) | Token: 34309_