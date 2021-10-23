# TLDR

- check reactandflow.md for main docs
- anythng here [should] strictly pertain to differences with flowtype

## quickies

```js
import React, { FC, ReactElement } from 'react';

export interface PropDef {
  readonly poop: string;
}
export const someFnComponent: FC<PropDef> = ({
  prop1,
  prop2,
}): ReactElement => {}
// ^ or one that doesnt accept props
// SomeComponent: FC<Record<string, never>> = (): ReactElement

export const fetchSomething (): Promise<PropDef> {
  return ...
}
const arrayOfObjects: PropDef[] = [propDef1, propDefX...s]
SomeEl = ({ }: SomeElProps): JSX.Element
```