<svelte:head>
    <title>svelte-interest-rate-equiv</title>
</svelte:head>

<script>
    let openingBalance
    let interestRate
    let repaymentsCount
    let endingBalance

    let openingBalanceValue
    let interestRateValue
    let repaymentsCountValue
    let endingBalanceValue

    let lastElem
    let altElem

    function newton_raphson_then_dichotomy(x, getY, derivative) {
        let higherY = +Infinity, lowerY = -Infinity
        let higherX, lowerX
        outer:
        while (true) {
            const xBak = x
            const m = derivative(x)
            const y = getY(x)
            if (y > 0) {
                if (y < higherY) {
                    higherY = y
                    higherX=x
                }
            } else if (y < 0) {
                if (y > lowerY) {
                    lowerY = y
                    lowerX=x
                }
            } else {
                break outer
            }
            x = x - y/m
            // if (higherX && lowerX && Math.abs(xBak - x) < Number.EPSILON * 10000) { //may have to increase 10000 to something larger to prevent freeze
            if (higherX && lowerX && Math.abs(xBak - x)) {
                let highX = higherX
                let lowX = lowerX

                while (true) {
                    let midX = (highX+lowX)/2
                    let midY = getY(midX)
                    if (midY > 0) {
                        if (highX === midX) {
                            x = midX
                            break outer
                        }
                        highX = midX
                    } else if (midY < 0) {
                        if (lowX === midX) {
                            x = midX
                            break outer
                        }
                        lowX = midX
                    } else {
                        x = midX
                        break outer
                    }
                }
            }
        }
    return x
    }

    function okLetsTry(event) {

        const empty = []
        for (const element of [openingBalance,interestRate,repaymentsCount,endingBalance]) {
            if (element.value === "") {
                empty.push(element)
            }
        }

        let curElem
        if (empty.length === 1) {
            if (empty[0] === event.target) {
                return
            }
            lastElem = empty[0]
            curElem = lastElem
            altElem = event.target
        } else if (empty.length !== 0 || !lastElem) {
            return
        } else {
            if (lastElem === event.target) {
                curElem = altElem
            } else {
                curElem = lastElem
                altElem = event.target
            }
        }

        switch (curElem) {
            case openingBalance:
                //not useful, for fun only
                openingBalanceValue = (endingBalanceValue*(interestRateValue + 1)**(-repaymentsCountValue)*((interestRateValue + 1)**repaymentsCountValue - 1))/(interestRateValue*repaymentsCountValue)
                break
            case interestRate:
                interestRateValue = newton_raphson_then_dichotomy(0.1, function getY(interestRateValue) {
                    return openingBalanceValue*interestRateValue*repaymentsCountValue*(1/((interestRateValue + 1)**repaymentsCountValue - 1) + 1) - endingBalanceValue
                }, function derivative(x) {
                    return openingBalanceValue*repaymentsCountValue*(-(repaymentsCountValue*x*(x + 1)**(repaymentsCountValue - 1))/((x + 1)**repaymentsCountValue - 1)**2 + 1/((x + 1)**repaymentsCountValue - 1) + 1)
                })
                break
            case repaymentsCount:
                //not useful, for fun only
                repaymentsCountValue = newton_raphson_then_dichotomy(25, function getY(repaymentsCountValue) {
                    return openingBalanceValue*interestRateValue*repaymentsCountValue*(1/((interestRateValue + 1)**repaymentsCountValue - 1) + 1) - endingBalanceValue
                }, function derivative(x) {
                    return (openingBalanceValue*interestRateValue*(interestRateValue + 1)**x*((interestRateValue + 1)**x - x*Math.log(interestRateValue + 1) - 1))/((interestRateValue + 1)**x - 1)**2
                })
                break
            case endingBalance:
                endingBalanceValue=((openingBalanceValue*interestRateValue)/((interestRateValue + 1)**repaymentsCountValue - 1)+interestRateValue*openingBalanceValue)*repaymentsCountValue
                break
        }
    }

    function handleKeydown(event) {
        const bak = event.target.value
        setTimeout(() => {
            if (event.target.value !== bak) {
                okLetsTry(event)
            }
        }, 1);
    }

</script>

<label for="openingBalance">openingBalance</label><input type="number" name="openingBalance" id="openingBalance" on:keydown={handleKeydown} on:change={okLetsTry} on:paste={event=>setTimeout(okLetsTry(event),1)} bind:this={openingBalance} bind:value={openingBalanceValue}>
<label for="interestRate">interestRate</label><input type="number" name="interestRate" id="interestRate" on:keydown={handleKeydown} on:change={okLetsTry} on:paste={event=>setTimeout(okLetsTry(event),1)} bind:this={interestRate} bind:value={interestRateValue}>
<label for="repaymentsCount">repaymentsCount</label><input type="number" name="repaymentsCount" id="repaymentsCount" on:keydown={handleKeydown} on:change={okLetsTry} on:paste={event=>setTimeout(okLetsTry(event),1)} bind:this={repaymentsCount} bind:value={repaymentsCountValue}>
<label for="endingBalance">endingBalance</label><input type="number" name="endingBalance" id="endingBalance" on:keydown={handleKeydown} on:change={okLetsTry} on:paste={event=>setTimeout(okLetsTry(event),1)} bind:this={endingBalance} bind:value={endingBalanceValue}>

<style>
</style>
