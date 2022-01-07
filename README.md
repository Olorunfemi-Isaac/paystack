def payment(request):
    plan = request.GET.get('sub_plan')
    fetch_membership = Membership.objects.filter(membership_type=plan).exists()
    if fetch_membership == False:
        return redirect('subscribe')
    membership = Membership.objects.get(membership_type=plan)
    price = float(membership.price)*100
    price = int(price)

    def init_payment(request):
        url = 'https://api.paystack.co/transaction/initialize/'
        headers = {
            'Authorization': 'Bearer ' +settings.PAYSTACK_SECRET_KEY,
            'Content-Type': 'application/json',
            'Accept': 'application/json',
            }
        datum = {
            "email": request.user.email,
            "amount": price,
        }
        x = requests.post(url, data=json.dumps(datum), headers=headers)
        if x.status_code != 200:
            return str(x.status_code)
            
        results = x.json()
        return results
    initialized = init_payment(request)
    print(initialized['data']['authorization_url'])
    print(initialized)
    amount = price/100
    instance = PayHistory.objects.create(amount=amount, payment_for=membership, user=request.user, paystack_charge_id=initialized['data']['reference'], paystack_access_code=initialized['data']['reference'])
    UserMembership.objects.filter(user=instance.user).update(reference_code=initialized['data']['reference'])
    link = initialized['data']['authorization_url']
    return HttpResponseRedirect(link)
    return render(request, 'app/payment.html')
