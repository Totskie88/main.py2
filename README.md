
from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.label import Label
from kivy.uix.button import Button
from kivy.uix.scrollview import ScrollView
from kivy.uix.popup import Popup
from kivy.metrics import dp, sp
from kivy.core.window import Window

# PINK THEME
PINK = (1.0, 0.72, 0.82, 1)
DARK_PINK = (0.85, 0.30, 0.52, 1)
LIGHT_PINK = (1.0, 0.90, 0.94, 1)
WHITE = (1, 1, 1, 1)
DARK = (0.25, 0.15, 0.20, 1)

Window.clearcolor = LIGHT_PINK


# FOOD MENU
MENU = [
    {
        "title": "MAIN MENU",
        "items": [
            ("Unlimited Tuslob Buwa", 70,
             "Minimum 3 persons", 3),

            ("Regular Set", 145,
             "Good for 2 persons", 1),

            ("TUSLOB BUWA SET (TAKEOUT)", 159,
             "Utok/Sabaw • 6 pcs Puso • Onion • Garlic", 1)
        ]
    },
    {
        "title": "ADD-ONS",
        "items": [
            ("Puso", 10, "Rice", 1),
            ("Lumpia", 10, "Crispy lumpia", 1),
            ("Dynamite", 12, "Spicy snack", 1),
            ("Ngohiong", 12, "Cebuano favorite", 1),
            ("Siomai", 10, "Steamed siomai", 1)
        ]
    },
    {
        "title": "DRINKS",
        "items": [
            ("Coke 1 Liter", 55, "Refreshing drink", 1),
            ("Coke Regular", 20, "Cold soft drink", 1),
            ("Bottled Water", 20, "Mineral water", 1)
        ]
    }
]


class TuslobBuwaApp(App):

    def build(self):
        self.title = "3REYA'S TUSLOB BUWA"
        self.cart = {}

        root = BoxLayout(
            orientation="vertical",
            padding=dp(12),
            spacing=dp(10)
        )

        # HEADER
        header = BoxLayout(
            orientation="vertical",
            size_hint_y=None,
            height=dp(105),
            padding=dp(8),
            spacing=dp(4)
        )

        header.add_widget(Label(
            text="[b]3REYA'S[/b]",
            markup=True,
            font_size=sp(30),
            color=DARK_PINK
        ))

        header.add_widget(Label(
            text="UNLIMITED TUSLOB BUWA",
            font_size=sp(17),
            bold=True,
            color=DARK
        ))

        header.add_widget(Label(
            text="Delicious food, happy moments!",
            font_size=sp(12),
            color=DARK_PINK
        ))

        root.add_widget(header)

        # MENU SCROLL AREA
        scroll = ScrollView(
            do_scroll_x=False
        )

        self.menu_layout = BoxLayout(
            orientation="vertical",
            spacing=dp(12),
            size_hint_y=None,
            padding=dp(4)
        )

        self.menu_layout.bind(
            minimum_height=self.menu_layout.setter(
                "height"
            )
        )

        for section in MENU:
            self.add_section(section)

        scroll.add_widget(self.menu_layout)
        root.add_widget(scroll)

        # CART AREA
        cart_box = BoxLayout(
            orientation="vertical",
            size_hint_y=None,
            height=dp(175),
            spacing=dp(5),
            padding=dp(8)
        )

        cart_box.add_widget(Label(
            text="YOUR SHOPPING CART",
            font_size=sp(17),
            bold=True,
            color=DARK_PINK,
            size_hint_y=None,
            height=dp(28)
        ))

        self.cart_label = Label(
            text="Your cart is empty.",
            font_size=sp(13),
            color=DARK,
            halign="left",
            valign="top",
            size_hint_y=1
        )

        self.cart_label.bind(
            size=self.update_cart_text_size
        )

        cart_box.add_widget(self.cart_label)

        self.total_label = Label(
            text="TOTAL: ₱0",
            font_size=sp(19),
            bold=True,
            color=DARK_PINK,
            size_hint_y=None,
            height=dp(30)
        )

        cart_box.add_widget(self.total_label)

        checkout_btn = Button(
            text="CHECKOUT",
            size_hint_y=None,
            height=dp(45),
            background_normal="",
            background_color=DARK_PINK,
            color=WHITE,
            bold=True
        )

        checkout_btn.bind(
            on_release=self.checkout
        )

        cart_box.add_widget(checkout_btn)

        root.add_widget(cart_box)

        self.update_cart()
        return root

    def update_cart_text_size(self, instance, size):
        instance.text_size = (size[0], None)

    def add_section(self, section):

        title = Label(
            text=section["title"],
            font_size=sp(20),
            bold=True,
            color=DARK_PINK,
            size_hint_y=None,
            height=dp(40)
        )

        self.menu_layout.add_widget(title)

        for name, price, description, minimum in section["items"]:

            card = BoxLayout(
                orientation="vertical",
                size_hint_y=None,
                height=dp(112),
                padding=dp(9),
                spacing=dp(3)
            )

            card.add_widget(Label(
                text=name,
                font_size=sp(15),
                bold=True,
                color=DARK,
                halign="left",
                size_hint_y=None,
                height=dp(25)
            ))

            card.add_widget(Label(
                text=description,
                font_size=sp(11),
                color=DARK,
                halign="left",
                size_hint_y=None,
                height=dp(23)
            ))

            bottom = BoxLayout(
                orientation="horizontal",
                spacing=dp(5)
            )

            if name == "Unlimited Tuslob Buwa":
                price_text = "₱70 / person"
            else:
                price_text = "₱" + str(price)

            bottom.add_widget(Label(
                text=price_text,
                font_size=sp(15),
                bold=True,
                color=DARK_PINK
            ))

            add_btn = Button(
                text="ADD +",
                size_hint_x=0.38,
                background_normal="",
                background_color=DARK_PINK,
                color=WHITE,
                bold=True
            )

            add_btn.bind(
                on_release=lambda btn, n=name, p=price,
                m=minimum: self.add_to_cart(n, p, m)
            )

            bottom.add_widget(add_btn)
            card.add_widget(bottom)

            self.menu_layout.add_widget(card)

    def add_to_cart(self, name, price, quantity):

        if name in self.cart:
            self.cart[name]["quantity"] += quantity
        else:
            self.cart[name] = {
                "price": price,
                "quantity": quantity
            }

        self.update_cart()

    def update_cart(self):

        if not self.cart:
            self.cart_label.text = "Your cart is empty."
        else:
            lines = []

            for name, item in self.cart.items():
                qty = item["quantity"]
                price = item["price"]
                subtotal = qty * price

                if name == "Unlimited Tuslob Buwa":
                    lines.append(
                        f"{name}: {qty} persons = ₱{subtotal}"
                    )
                else:
                    lines.append(
                        f"{name} x{qty} = ₱{subtotal}"
                    )

            self.cart_label.text = "\n".join(lines)

        total = sum(
            item["price"] * item["quantity"]
            for item in self.cart.values()
        )

        self.total_label.text = f"TOTAL: ₱{total}"

    def checkout(self, instance):

        if not self.cart:
            message = Label(
                text="Your cart is empty!",
                color=DARK,
                font_size=sp(16)
            )

            Popup(
                title="3REYA'S",
                content=message,
                size_hint=(0.8, 0.3)
            ).open()

            return

        total = sum(
            item["price"] * item["quantity"]
            for item in self.cart.values()
        )

        summary = []

        for name, item in self.cart.items():
            qty = item["quantity"]
            subtotal = item["price"] * qty
            summary.append(
                f"{name} x{qty} = ₱{subtotal}"
            )

        summary.append("")
        summary.append(f"TOTAL: ₱{total}")

        content = BoxLayout(
            orientation="vertical",
            padding=dp(12),
            spacing=dp(10)
        )

        content.add_widget(Label(
            text="\n".join(summary),
            color=DARK,
            font_size=sp(14)
        ))

        content.add_widget(Label(
            text="Thank you for ordering at 3REYA'S!",
            color=DARK_PINK,
            font_size=sp(14),
            bold=True
        ))

        close_btn = Button(
            text="CONFIRM ORDER",
            size_hint_y=None,
            height=dp(48),
            background_normal="",
            background_color=DARK_PINK,
            color=WHITE
        )

        popup = Popup(
            title="ORDER SUMMARY",
            content=content,
            size_hint=(0.9, 0.75)
        )

        def confirm_order(btn):
            self.cart.clear()
            self.update_cart()
            popup.dismiss()

            Popup(
                title="ORDER PLACED!",
                content=Label(
                    text="Thank you for choosing 3REYA'S!",
                    color=DARK,
                    font_size=sp(16)
                ),
                size_hint=(0.85, 0.35)
            ).open()

        close_btn.bind(
            on_release=confirm_order
        )

        content.add_widget(close_btn)
        popup.open()


if __name__ == "__main__":
    TuslobBuwaApp().run()
    
