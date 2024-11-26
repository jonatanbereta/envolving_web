
# Question 1 (short snippet of code)

The short snippet code presented, named `MemberRenewals.vue`,  is part from a project I delivered to Projaide (https://projaide.com/), a non-profit organization in Montreal. The project consists of a system for managing the organization's social programs and managing the beneficiary families.

## Technologies Used for the project

- Laravel
- VueJS
- Tailwind
- MySQL
- Docker

## Explanation

- The file, named `MemberRenewals.vue`, presents a table that lists the renewal history of a beneficiary from the food bank. 
- This table is part of a Tab on the MemberProfile page. It will receive a Member model as a prop, which will bring with it a field containing the registration history.
- Each item in the history contains a status, whether the registration is active or needs renewal.
- Below the table, there is a button where the user can make a renewal, which will only be possible if the registration has expired in the current period. 
- If renewal is allowed, a payment registration component, `PaymentRegister.vue`, is invoked. 
- In it, the information regarding the renewal price of the social program, as well as the payment method used, must be recorded. 
- Finally, upon completing the registration, the history is immediately updated with the new renewal with valid status. 

This component was developed out of a need to make the beneficiary renewal process more practical and easier for the user to understand, as when renewing, it is possible to register the payment without leaving the screen. Thus, the user can check the history, make the renewal, and register the payment in the same process with a few clicks.


## Contributing
This project was developed by following the organization's actions weekly. I had the opportunity to listen to each user about what they would like to have in a system to improve their work, including which features and resources should be present. The result after delivery was that the food distribution process, registrations, and reports are now faster, more practical, and more efficient.



