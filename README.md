# Railway-Reservation-System
class RailwayReservationSystem:
    def _init_(self, total_seats):
        self.total_seats = total_seats
        self.available_seats = [True] * total_seats  # Array for seat availability
        self.reservations = ReservationQueue()  # Queue for ticket bookings
        self.waitlist = Waitlist()  # Linked List for waitlisted passengers
        self.undo_stack = UndoStack()  # Stack for undoing bookings

    def show_available_seats(self):
        print("\n Seat Availability:")
        available = [i + 1 for i, seat in enumerate(self.available_seats) if seat]
        if available:
            print("Seats Available:", available)
        else:
            print(" No seats available. Waitlist is active.")

    def book_ticket(self, passenger_name):
        for i in range(self.total_seats):
            if self.available_seats[i]:  # Find first available seat
                self.available_seats[i] = False  # Mark seat as booked
                self.reservations.enqueue((passenger_name, i + 1))  # Add to queue
                self.undo_stack.push((passenger_name, i + 1))  # Save for undo
                print(f" Ticket booked for {passenger_name} (Seat {i + 1})")
                return
        # No available seats, add to waitlist
        self.waitlist.add_passenger(passenger_name)
        print(f" No available seats. {passenger_name} added to waitlist.")

    def cancel_ticket(self, passenger_name):
        cancelled_seat = self.reservations.cancel_booking(passenger_name)
        if cancelled_seat:
            self.available_seats[cancelled_seat - 1] = True  # Free the seat
            print(f" Ticket for {passenger_name} (Seat {cancelled_seat}) cancelled.")
            if not self.waitlist.is_empty():
                waitlisted_passenger = self.waitlist.remove_passenger()
                self.book_ticket(waitlisted_passenger)  # Assign freed seat
        else:
            print(f" No booking found for {passenger_name}.")

    def undo_last_booking(self):
        last_booking = self.undo_stack.pop()
        if last_booking:
            passenger_name, seat_number = last_booking
            self.available_seats[seat_number - 1] = True  # Free the seat
            self.reservations.cancel_booking(passenger_name)  # Remove from queue
            print(f" Undo: {passenger_name}'s booking (Seat {seat_number}) removed.")
        else:
            print(" No bookings to undo.")

    def show_waitlist(self):
        self.waitlist.display_waitlist()

    def show_bookings(self):
        self.reservations.display_bookings()


class ReservationQueue:
    def _init_(self):
        self.queue = []

    def enqueue(self, reservation):
        self.queue.append(reservation)  # Add booking (FIFO)

    def cancel_booking(self, passenger_name):
        for i, (name, seat) in enumerate(self.queue):
            if name == passenger_name:
                del self.queue[i]  # Remove from queue
                return seat  # Return freed seat number
        return None

    def display_bookings(self):
        if not self.queue:
            print(" No bookings made yet.")
            return
        print("\n Current Bookings:")
        for name, seat in self.queue:
            print(f" {name} - Seat {seat}")


class Waitlist:
    def _init_(self):
        self.head = None

    def add_passenger(self, passenger_name):
        new_node = WaitlistNode(passenger_name)
        if not self.head:
            self.head = new_node
        else:
            temp = self.head
            while temp.next:
                temp = temp.next
            temp.next = new_node

    def remove_passenger(self):
        if not self.head:
            return None
        passenger_name = self.head.passenger_name
        self.head = self.head.next  # Move head to next passenger
        return passenger_name

    def is_empty(self):
        return self.head is None

    def display_waitlist(self):
        if not self.head:
            print(" No passengers on the waitlist.")
            return
        print("\n Waitlist:")
        temp = self.head
        while temp:
            print(f" {temp.passenger_name}")
            temp = temp.next


class WaitlistNode:
    def _init_(self, passenger_name):
        self.passenger_name = passenger_name
        self.next = None


class UndoStack:
    def _init_(self):
        self.stack = []

    def push(self, booking):
        self.stack.append(booking)

    def pop(self):
        return self.stack.pop() if self.stack else None


# Main program execution
system = RailwayReservationSystem(total_seats=5)

while True:
    print("\n Welcome to Railway Reservation System")
    print("1. View Available Seats")
    print("2. Book Ticket")
    print("3. Cancel Ticket")
    print("4. Undo Last Booking")
    print("5. View Bookings")
    print("6. View Waitlist")
    print("7. Exit")

    choice = input("Enter your choice: ")

    if choice == "1":
        system.show_available_seats()
    elif choice == "2":
        name = input("Enter passenger name: ")
        system.book_ticket(name)
    elif choice == "3":
        name = input("Enter passenger name to cancel: ")
        system.cancel_ticket(name)
    elif choice == "4":
        system.undo_last_booking()
    elif choice == "5":
        system.show_bookings()
    elif choice == "6":
        system.show_waitlist()
    elif choice == "7":
        print(" Exiting... Thank you for using the Railway Reservation System!")
        break
    else:
        print(" Invalid choice! Please try again.")
