<script lang="ts">
    import { addDoc, collection, serverTimestamp } from 'firebase/firestore';
    import { db, sendEmail } from '../firebaseConfig';
    import { user } from '../stores';
    import Swal from 'sweetalert2';
    import { toast, modal } from '../utils/swal';

    async function addSeller() {
        const form = await modal.fire({
            title: `Dodaj nowego sprzedawcę`,
            html: `<form><input class="swal2-input" placeholder="Imię" name="firstName" data-form-type="other"><input class="swal2-input" placeholder="Nazwisko" name="lastName" data-form-type="other"><input class="swal2-input" placeholder="Klasa" name="classSymbol" data-form-type="other"><input class="swal2-input" placeholder="Email" name="email" data-form-type="other"></form>`,
            confirmButtonText: 'Dodaj',
            didOpen: () => (<HTMLInputElement>Swal.getPopup().querySelector('form')[0]).focus(),
            preConfirm: async () => {
                const form = Swal.getPopup().querySelector('form');
                const firstName = (<HTMLInputElement>form.firstName).value;
                const lastName = (<HTMLInputElement>form.lastName).value;
                const classSymbol = (<HTMLInputElement>form.classSymbol).value;
                const email = (<HTMLInputElement>form.email).value || null;

                if (!firstName || !lastName || !classSymbol) return Swal.showValidationMessage(`Wypełnij wszystkie pola`);
                if (email && !/[\w-\.]+@([\w-]+\.)+[\w-]{2,4}/.test(email)) return Swal.showValidationMessage(`Niepoprawny adres email`);

                return { firstName, lastName, classSymbol, email };
            },
        });
        if (!form.isConfirmed) return;

        const { firstName, lastName, classSymbol, email } = <SellerDataForm>form.value;
        const sellerDoc: SellerDocument = {
            firstName,
            lastName,
            classSymbol,
            email,
            creator: {
                uid: $user.uid,
                email: $user.email,
            },
            createdAt: serverTimestamp(),
        };
        addDoc(collection(db, 'sellers'), sellerDoc);

        toast.fire({
            icon: 'success',
            title: `Dodabno sprzedawcę`,
            text: `${firstName} ${lastName} ${classSymbol}`,
            timer: 4000,
        });

        if (email)
            sendEmail({
                to: email,
                subject: 'Witaj na kiermaszu!',
                html: `Cześć ${firstName},<br><br>Witamy na kiermaszu podręczników używanych w mechaniku!<br>Będziesz otrzymywać zautomatyzowane wiadomości za każdym razem gdy któryś z twoich podręczników zostanie sprzedany.<br>Wszelkie pytania kierować możesz na adres email kiermasz@mechaniktg.pl lub osobiście w bibliotece.<br><br>Pozdrawiamy,<br>Biblioteka ZSTiO`,
            });
    }
</script>

<button on:click={addSeller} aria-label="dodaj sprzedawcę" class="btn btn-hover">
    <svg xmlns="http://www.w3.org/2000/svg" height="1em" viewBox="0 0 640 512">
        <path
            d="M96 128a128 128 0 1 1 256 0A128 128 0 1 1 96 128zM0 482.3C0 383.8 79.8 304 178.3 304h91.4C368.2 304 448 383.8 448 482.3c0 16.4-13.3 29.7-29.7 29.7H29.7C13.3 512 0 498.7 0 482.3zM504 312V248H440c-13.3 0-24-10.7-24-24s10.7-24 24-24h64V136c0-13.3 10.7-24 24-24s24 10.7 24 24v64h64c13.3 0 24 10.7 24 24s-10.7 24-24 24H552v64c0 13.3-10.7 24-24 24s-24-10.7-24-24z"
        />
    </svg>
    Dodaj Sprzedawcę
</button>

<style>
    button {
        position: sticky;
        top: 1rem;
        display: flex;
        align-items: center;
        gap: 0.75rem;
        /* font-size: clamp(0.75rem, 3.75vw, 1.25rem); */
        font-size: 1.25rem;
        font-weight: 900;
        transition: background-color 75ms ease;
    }
</style>
